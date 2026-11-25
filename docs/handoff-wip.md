# Handoff: victory-fork merge in progress (work in progress)

Continue on the other PC (RTX 4070 Ti). Read this file first.

## 1. State

- Branch `sync-upstream` holds an **unfinished merge** of upstream `Marvisak/VIctory` `main` (315 commits) into this fork. It is committed as a merge commit that does not compile yet; keep working on this branch and commit the fixes on top.
- Pull both branches: `git fetch origin; git switch sync-upstream`; the heap-limit work is on `fix/view-heap-limit` (WIP commit).
- All conflicts are resolved and staged. `victory-game` compiles. `victory-shader` has about 36 compile errors left (see the state files below). `victory-gpu` has not been checked yet.
- The GPU-side fork files were taken from upstream whole: `draw.rs`, `host.rs`, `descriptor/host.rs`, `shadow.rs`, `texture.rs`, `watch.rs`, `resource.rs`, `indirect.rs`, `memory.rs`, `pipeline/*.rs`, `capture.rs`, `copystat.rs`. The fork's originals are readable with `git show a803c87:<path>`.

## 2. Agent state files (copied verbatim)

### p6 — merge shader files A

# p6 state (paused)

Repo: victory-fork, merge of victory/main into sync-upstream. My files:
lower.rs, shader.rs, wave.rs, bitcode.rs, waterfall_tests.rs.

## Done
- All five files: no conflict markers, brace-balanced, `git add`ed (lower.rs re-added after later edits).
- shader.rs: upstream names (ATOMIC64_TYPED/HEAP, gs, memory handles). Fork's Geometry struct and
  geometry field removed. Per user instruction: fork's view-heap clamp (VIEW_LIMIT, set_view_heap_limit,
  view_heap_limit, and the limit/2047 branches in heap_handle) REMOVED; upstream HEAP_VIEWS clamp kept.
- wave.rs: upstream Half path + fork's Red64F/ActiveF64, buddies check moved into half(); half-sum test
  normalises Shape::Half. Not yet compiled/tested.
- lower.rs: markers resolved (mostly upstream where both sides rewrote a feature). Brace fixes done.
  Replaced upstream-wholesale: interp, image, buffer, emit_indices. Dropped fork's intersect module
  (`mod intersect` removed; upstream lower/bvh.rs kept). Removed duplicate test
  buffer_cache_invalidations_lower_as_no_ops. Added Interp::Weighted => Err arm (fork's Weighted
  handling NOT ported: regression to fix).

## Last cargo check
- victory-shader --all-targets: about 13 error lines, including lower.rs:12596 (one
  `packed_half(&sig, &[(8, None)], 8, 0)` call still with 4 args; needs 3 args `(&sig, 8, 0)`).

## Open issues for next step
1. lower.rs:12596 fix call to 3 args (lines 12560/12587 already fixed).
2. p7 report: shader.rs view-heap fns missing. CONFLICT: the user's instruction was to DROP
   view_heap_limit machinery; emit/mod.rs:221 still re-exports set_view_heap_limit/view_heap_limit and
   victory-gpu / apps main.rs call them. Needs user decision or mod.rs owner to remove the re-export and
   callers, not restore the fork code.
3. lower.rs:12007 `*slot` (test in crate::native::tests) — check emit_indices test area.
4. intersect.rs / intersect_tests.rs (fork files, not mine) are now orphaned: intersect_tests imports
   super::lower::intersect::TEST_MEMORY and super::host; owner must delete or port them.
5. wave.rs one error reported by p7 (see last cargo check output).
6. Fork-only features dropped or at risk (need tests): fork packed_layout ("behind" placement),
   half_outputs partner info, memory_flat/memory_smem fork path, descriptor_phi/slot_stride, root_bias
   (USER_VIEW_BIAS), view_address words branch, computed_with, Interp::Weighted interpolation,
   constant-lane-mask select arm, uniform_test mesh path, intersect.rs BVH (replaced by upstream bvh).
7. Once it compiles: `cargo test -p victory-shader` and fix failures in my files.

## Next step
Finish item 1, then re-run cargo check, then address items 3, 5, 6.

### p7 — merge files B (structure, rect, Cargo)

# p7 state (paused)

## Repo
C:\Users\zhang\OneDrive\Documents\github\victory-fork, branch sync-upstream, merge of victory/main in progress. Nothing committed. Do not run git merge/commit/reset/checkout/stash.

## Files done (no conflict markers, git add'ed)
- apps/victory/src/main.rs, apps/victory/src/translate.rs
- crates/victory-game/src/sgd5.rs, crates/victory-game/Cargo.toml (miniz_oxide added)
- crates/victory-loader/Cargo.toml
- crates/victory-hle/src/modules/{xmem,dstorage,gameinput}.rs
- crates/victory-shader/src/emit/{mod,container,diff_tests,rect,valu_tests}.rs
- crates/victory-shader/src/emit/host.rs (root_indexed_views restored)
- crates/victory-shader/src/{io,ir,native,resource,structure,lib}.rs
- crates/victory-shader/Cargo.toml (miniz_oxide added)
- Cargo.toml (miniz_oxide = "0.9.1" added), Cargo.lock (updated by cargo check)
- structure.rs: all diff3 base markers removed; fork meet_in/meet_within renamed meet_in_wide/meet_within_wide; upstream meet/meet_in/decides kept.

## Files in progress
- None mid-edit. Last edit finished: gated `intersect_tests` in emit/mod.rs with cfg(all(test, windows)).

## Cargo check status
- `cargo check -p victory-game --all-targets`: PASSES.
- `cargo check -p victory-shader --all-targets`: 36 error lines. Last run summary by file:
  - emit/lower.rs (p6): 10 (Interp::Weighted arm missing at 9843; `slot` not in scope at 12007; 3-arg fn called with 4 args at 12560/12587/12596)
  - emit/shader.rs (p6, cause of mod.rs:222 error): set_view_heap_limit / view_heap_limit missing
  - emit/intersect_tests.rs: TEST_MEMORY missing from lower::intersect (p6); HostShader literal missing fields (line 181)
  - emit/memory_tests.rs (2), stride_tests.rs, synthetic_tests.rs, bvh_tests.rs, semantics_tests.rs, wave64_tests.rs: HostShader literals missing root_indexed_views and/or memory/heap fields. NOT my files; these are upstream test files that need the fields added.
  - emit/wave.rs (p6): 1
  - emit/mod.rs:222: unresolved set_view_heap_limit/view_heap_limit (depends on p6's shader.rs)
- Mine (emit/mod.rs, io.rs, native.rs, ir.rs, resource.rs, structure.rs, host.rs, valu_tests.rs): no remaining errors reported except mod.rs:222 above.

## Messages sent
- p6: lower.rs / shader.rs / wave.rs / intersect_tests errors (listed above).
- p5: earlier reports on rect port, structure, Cargo.

## Next step
1. Owner of emit/shader.rs (p6) restores set_view_heap_limit / view_heap_limit from HEAD's shader.rs lines ~118-132, and fixes lower.rs and wave.rs.
2. Add root_indexed_views (and memory/heap fields where missing) to the HostShader literals in memory_tests.rs, stride_tests.rs, synthetic_tests.rs, bvh_tests.rs, semantics_tests.rs, wave64_tests.rs, intersect_tests.rs (ask p5 who owns them).
3. Re-run `cargo check -p victory-shader --all-targets` with CARGO_TARGET_DIR=C:\Users\zhang\.cargo-target\victory-fork, then the marker grep:
   git grep -n -E '^(<<<<<<<|\|\|\|\|\|\|\||=======$|>>>>>>>)' -- crates/victory-shader   (must print nothing)
4. Not compiled yet: victory-gpu callers of rect_list_gs / pipeline_with_gs need renaming (rectangle_gs, pipeline_gs).

### p8 — heap-limit fix

# p8 state (paused for terminal restart)

## Task
Implement the view-heap-limit fix as an upstream PR branch, in
C:\Users\zhang\victory-pr-heap (branch fix/view-heap-limit, based on upstream main).
Work ONLY there (p9 uses victory-base). Do not commit. Env:
PATH=%USERPROFILE%\.cargo\bin;... and CARGO_TARGET_DIR=C:\Users\zhang\.cargo-target\victory-pr-heap

Diagnosis (already sent to p5): on this Intel Arc, HostHeap::new cannot make the
2^21 view heap and falls back to ~65,540 slots, but shaders still clamp to 2^21-1
(out-of-bounds on hardware). Fix: probe the real size once, set the shader clamp
from it, size the heap from it, add it to the shader cache key.

## Done so far (worktree is still clean apart from these)
- crates/victory-shader/src/emit/shader.rs:
  - NULL_BUFFER_VIEW const REMOVED; replaced by `pub fn null_buffer_view() -> u32 { super::view_heap_limit() - 2 }`
  - HEAP_VIEWS const kept (default 1<<21); HEAP_SAMPLERS kept
  - line ~459 sampler/view clamp now uses `super::view_heap_limit()`
  - Both edits complete; file is consistent.

## Left to do
1. victory-shader/src/emit/mod.rs: add near loop_limit():
   `static VIEW_HEAP_LIMIT: OnceLock<u32>`; `pub fn view_heap_limit() -> u32` (get_or_init -> HEAP_VIEWS);
   `pub fn set_view_heap_limit(views: u32) -> bool` (OnceLock::set(...).is_ok()).
   Update re-export at mod.rs:206 (NULL_BUFFER_VIEW -> null_buffer_view; add view_heap_limit/set_view_heap_limit).
   Update doc lines 85-96 that mention HEAP_VIEWS / NULL_BUFFER_VIEW.
2. victory-shader/src/emit/lower.rs:4956 `super::HEAP_VIEWS - 1` -> `super::view_heap_limit() - 1`;
   lower.rs:4969 `super::NULL_BUFFER_VIEW` -> `super::null_buffer_view()`.
3. victory-shader/src/emit/host.rs:1368 `super::HEAP_VIEWS as usize` -> `view_heap_limit()`;
   host.rs:1698 `super::NULL_BUFFER_VIEW as usize` -> `null_buffer_view() as usize`; doc at host.rs:1216.
4. victory-gpu/src/descriptor/host.rs:
   - add `probe_view_limit()` (Once; descending sizes [1<<21, PORTABLE_VIEWS, 1<<19, 1<<18, 1<<17, 1<<16];
     first that CreateDescriptorHeap accepts -> set_view_heap_limit; log info with the size; warn if none).
   - HostHeap::new: call probe_view_limit() when !sampler; view heap size = views (the probed value);
     room = views-2; extra = DYNAMIC if count+DYNAMIC<=room else MIN_DYNAMIC.min(room); count = count.min(room-extra) (warn if clipped);
     make(views). strides buffer uses views; pairs_end and null checks use null_buffer_view().
   - write_stride (~line 457): use view_heap_limit() instead of HEAP_VIEWS.
   - test at ~1422: NumDescriptors >= view_heap_limit().
5. victory-gpu/src/shader_cache.rs: build()/key() include view_heap_limit() (key gets a new param; update the
   key test at ~line 430-450 call sites). Also call probe_view_limit() at top of build() and host_shader().
6. victory-gpu/src/pipeline.rs translate() and pipeline/library.rs tag(): call probe_view_limit() first;
   tag() also hashes view_heap_limit() (same reason as the cache key).
7. Checks (all must pass, output in report):
   a) cargo test -p victory-gpu --lib -- descriptor::host::tests memory::tests --test-threads=1
   b) cargo test -p victory-shader --lib --test-threads=1 (no NEW failures; known: fspx, image_atomics_of_64_bits, heap_buffer_read)
   c) cargo clippy --workspace --all-targets --locked -- -D warnings  and  same with --all-features
   Use cmd with vcvars64 if link fails: scratch chk.bat pattern (SDK install status: p5 said it was installing).
8. Report to p5: diffstat, the probe log line (size this GPU got), check output.

## Last check results
- Base tree (victory-base, upstream): both target tests FAIL (descriptor/host.rs:1422 heap < HEAP_VIEWS; memory.rs:673 "past the dynamic slots").
- PR worktree: no checks run yet.

## Next step
Finish item 1 (mod.rs), then 2-3 (shader call sites), then 4-6, then run check 7a first.

### p9 — test diagnosis

# p9 findings (paused)

Base worktree C:\Users\zhang\victory-base: temp edits reverted (git checkout -- .), status clean.

## Test 3: fspx_programs_run_with_the_hardwares_dot_products (valu_tests.rs:1619)
- WARP leg alone PASSES, including the sel_hi_lo case (opsel 2, opsel_hi 5, inline 244 = 2.0).
- Hardware (Intel Arc) gives 0x7FFFFF70, expected 0x7FFFFEB0 (diff 0xC0) on that case.
- So our expectation matches WARP; the hardware result differs. Hand-computation of the
  expected value not finished. Conclusion so far: the divergence is hardware-specific
  (translator output or Intel driver), not a wrong expectation.
- Minimal fix: skip only that case on hardware (comment), keep the others.

## Test 2: a_heap_buffer_read_where_no_buffer_view_is_reads_zeros (stride_tests.rs:304)
- Debug layer unavailable on this PC (CreateDeviceDebugLayer / D3D12GetDebugInterface
  returns 0x887A002D: Graphics Tools optional feature missing). No debug messages.
- Bisect: fails even with no root view bound (variant B) and with a valid stride
  (variant C). So it is not the root view binding.
- Real cause found: the test's View::NullBuffers makes a descriptor heap of
  HEAP_VIEWS = 1<<21 = 2,097,152 descriptors (host.rs dispatch, heap closure ~1350).
  CreateDescriptorHeap returns E_INVALIDARG ("parameter is incorrect") for it on this GPU.
- Minimal fix idea: size the heap to what the GPU allows (query
  D3D12_FEATURE_D3D12_OPTIONS resource binding tier / max descriptors), or skip the test
  when the heap cannot be created. Not yet tested.

## Test 1: image_atomics_of_64_bits (valu_tests.rs:1426)
- Missing capability: AtomicInt64OnTypedResourceSupported false.
- Fix: skip when the OPTIONS9 cap is false (same pattern as "no SM 6.6 ... skipped").

## Next steps
- Confirm test 2 by changing only the heap size (temp, in base) and rerun.
- Hand-compute the dot4 sel_hi_lo expected value vs WARP.


## 3. Branch `fix/view-heap-limit` (for an upstream PR)

Based on upstream `main`. It fixes the GPU texture-heap size: the heap size is probed on the GPU instead of assumed to be 2^21. Work in progress; see the heap-limit section (p8 above).

## 4. Plan

- **M1** Merge compiles, then tests and clippy match the upstream baseline. Five GPU-bound tests fail on Intel Arc: `descriptor::host` dynamic_slots, `memory` lists_share, `stride` heap_buffer_read, `image_atomics_of_64_bits`, `fspx` dot products.
- **M2** Heap-limit fix and test capability skips, as upstream PRs.
- **M3** Port the dropped fork switches (library map/trim, LATE_WAIT/RELAY, CARD_BUFFERS, INPLACE, draw.rs switches).
- **M4** Re-enable the lints, README.
- **M5** Needs the game and a strong GPU: ray-traced lighting, fps, memory.

## 5. Setup on the new PC

1. Install rustup, the Visual Studio Build Tools with the C++ workload, and the Windows SDK.
2. Set `CARGO_TARGET_DIR` to a folder outside OneDrive (for example a `.cargo-target\victory-fork` folder under your user profile).
3. `git fetch` and `git checkout sync-upstream`, then continue from section 1.
