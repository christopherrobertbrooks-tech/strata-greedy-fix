# strata-greedy-fix

**Strata 0.1.45 and 0.2.0-beta: a request with `temperature: 0` is not greedy, and on Qwen3.6 the MTP drafts never run.**

Since Strata 0.1.45, the server fills in the model card's sampling for anything a request leaves out. A request that
asks for `temperature: 0` keeps temperature 0, but still gets the card's `presence_penalty: 1.5` (Qwen3.6 always;
Qwen3.8-Flash-Next when not thinking). With a penalty, the engine samples from the penalised logits, so the answer is
not the greedy one. On the Qwen3.6 engine the MTP drafts are also switched off: `drafts accepted 0 of 0` on every
temperature-0 request. Reported upstream in [Niko1221/Strata#2940](https://github.com/Niko1221/Strata/issues/2940);
the fix below was [PR #3119](https://github.com/Niko1221/Strata/pull/3119), which was closed in the 0.2.0 tracker reset.

## Quick workaround (no code change)

Add `"sampling": {"presence_penalty": 0}` to your Strata config. Note that this turns the card's penalty off for
*every* request, not just greedy ones.

## The fix

When the request, the shared Chat settings or the config's `sampling` block (the first that sets a temperature) asks
for temperature 0, the card's penalties are left out. Everything else is unchanged.

```
cd <your Strata folder>
git apply strata-0.2.0-beta.patch     # or strata-0.1.45.patch
```

## Measured (Tesla V100-PCIe-32GB, Qwen3.6-35B-A3B UD-Q4_K_XL, 0.2.0-beta)

| | answer = true greedy | drafts accepted | prose | code |
| --- | --- | ---: | ---: | ---: |
| 0.2.0-beta | no | 0 of 0 | 114 tok/s | 112 tok/s |
| with the fix | **yes** | 115/166, 260/280 | **166** | **194** |

From 16K tokens up, the drafts cost a little on this card (11.6 vs 14.6 tok/s at 114K). That's MTP's own trade-off,
the same as before 0.1.45. All 52 of Strata's server test files give the same results with and without the fix.

## Files

- `strata-0.2.0-beta.patch`, `strata-0.1.45.patch`: the change (server, its tests, one sentence in `docs/DETAILS.md`),
  for each version.
- `results/`: the runs behind the table (`*-run.log`) and every server test file's result without and with the fix
  (`*-tests-before.txt`, `*-tests-after.txt`).

Strata is © its authors, MIT licence; these patches are under the same licence. Made with AI assistance (Claude).
