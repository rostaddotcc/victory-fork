# Integration branch, 25 November

Base: `9376fd6` ("Frame rate, processor side…"), the head of `fxceless/victory-fork` at the
time of writing. This branch is **ours, for testing** — it exists so that the sensible open pull
requests can be run together on a real machine instead of one at a time, and so that a broken
combination is found here rather than in someone else's tree.

Nothing here is ours except the merge work, the conflict resolutions and the test runs.

## Included

| | what it does | state upstream |
|---|---|---|
| #175 | scopes the title's memory-mapping lock to the affected ranges | open, `dirty` |
| #177 | batches the 64 KiB block queries in `texture mapped_bytes` | open, `unstable` |
| #374 | keeps all 32 indices in the first BC7 dictionary entry | open, `unstable` |
| #349 | `v_div_fixup_f32`: 0.0/x and x/Inf no longer become infinities | open, `unstable` |
| #355 | signed flag on `wavePrefixOp.f32`; `v_cvt_{u16,i16}_f16` test NaN in integer arithmetic | open, `dirty` |
| #353 | MIMG 3D Z texel offsets, a16 offset truncation, cubemap `sample_d` LOD | open, `dirty` |

## Conflict resolutions

- **#175**: one hunk, the re-export list in `crates/victory-gpu/src/lib.rs`. `main` had added
  `title_memory_reading_now`; the PR adds `title_memory_reading_range` and
  `title_memory_remapping_range`. Kept the union of all of them.
- **#355 and #353**: their production hunks (`lower.rs`, `host.rs`) applied with
  `git apply -3`; their **test** additions in `valu_tests.rs` (144 and 292 lines) conflict with
  tests merged since and are **not** included. The two shader fixes are therefore untested by
  their own tests here; only `victory-shader`'s existing 171 tests cover them.

## Left out on purpose

- **#358** (pages a placement locked) — conflicts structurally with the per-frame `STATS` array
  that `main` added to `inplace.rs`. A hand-merge there could not be verified, so the author
  should rebase it. Not attempted.
- **#282, #386, #396, #11** — closed upstream, i.e. already handled by the maintainer's own work.

## What was measured

With Wine/vkd3d-proton serving `d3d12` and Microsoft's `d3dcompiler_47`
(`WINEDLLOVERRIDES="d3d12=n;d3d12core=n;dxgi=n;d3dcompiler_47=n"`):

- `victory-gpu`: **147 passed, 1 failed, 3 ignored** — the failure is the known platform one
  (`pipeline::library::tests::a_rejected_blob_gives_an_empty_library`, vkd3d-proton refuses the
  garbage blob where D3D12 returns an empty library). `main` alone is at 130 passed, so the five
  PRs bring 17 tests with them.
- `victory-formats`: 26 passed, 4 ignored.
- `victory-shader` (native Linux): 171 passed, 2 ignored.
- Release and diag profiles both build (`cargo xwin build -p victory`, `--profile diag --features
  diagnostics`).

Game runs have not been done on this branch yet. On this rig a single run proves nothing: three
runs minutes apart, two of them the same binary, differ by 2–4x at matched draw counts, so the
protocol is A/B/A/B with medians across runs, never one run against one run.
