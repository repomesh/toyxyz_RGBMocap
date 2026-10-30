# Source package validation

The Git-based distribution was tested on the Windows PC listed in the
[user guide](USER_GUIDE.md#install). It has not been tested on a clean Windows
installation or on other GPUs or drivers.

## Recorded checks

- A separate local Git checkout installed all six pinned runtime profiles using
  `install.cmd`. Blender and FFmpeg were downloaded and hash checked; previously
  verified package downloads were reused. Installation took approximately 10 min
  32 sec on the tested PC. CMD launch and launch-error reporting were checked.
- ExMR, FlowHMR and GVHMR each processed a one-second video, including inference,
  correction, rig generation and FBX export. Thirteen worker fingerprint inputs
  were checked. This does not establish long-video performance on other systems.
- The included correction module and a developer rebuild produced identical
  root positions and local rotation values on the tested clips: 30 frames for
  each model and 210 additional frames for ExMR and FlowHMR, with automatic,
  manual and disabled foot contact. Other motions and platforms were not covered.
- One Mixamo FBX, one VRM and one PMX character were imported and retargeted.
  Mesh-inclusive and animation-only FBX files were reimported into Blender; bone
  animation and mesh options were present without armature-object transform keys.
  A PMX VMD was reimported with Blender MMD Tools. Visual equivalence in the
  standalone MMD application was not established by that check.
- A local Git update retained test model and project data and relaunched the GUI;
  runtime verification took approximately 1 min 55 sec. Twenty automated tests
  covered simulated download errors, unsafe archive paths, update guards and
  interrupted replacement. Actual power loss and disk exhaustion were not tested.
- The public source snapshot was checked for forbidden generated/model payloads,
  and its manifest hashes matched. Pattern checks found no typical private-path or
  credential formats in scanned text; such scans cannot prove absence of secrets.

## Latest published-source audit (2026-10-27)

The audit started from Git commit `1daf49c` and used the existing installed
runtimes and legitimately acquired local model files. All test scripts and
outputs were kept outside the public checkout.

- 150 targeted regression checks passed for project saving, model selection,
  character handling, export options, runtime guards and update transactions.
- Each of the three AI models processed 30 new frames through correction and FBX
  export. Blender imported all three 52-bone source rigs with `Hips` as root;
  sampled integer and fractional animation frames contained finite transforms.
- Mixamo FBX, VRM and PMX imports, retargeting and six character FBX exports
  passed. Reimported FBX rest-joint positions matched the imported character's
  original rest pose within 0.1 mm; no armature-object transform keys were found.
  The PMX VMD also loaded with Blender MMD Tools.
- The source checker was corrected to accept the approved README samples and
  root agent guide, ignore untracked local model data, and still reject model
  files accidentally tracked by Git. Archive and Git-checkout cases were tested.
- Local documentation links resolved. The source manifest was refreshed after
  the small documentation and checker fixes.

This audit did not repeat fresh installation, long-clip performance testing,
interactive playback, standalone MMD playback or testing on other GPUs.

## Character preview and long-path fixes (2026-10-27)

- Final preview windows no longer reload themselves. Loader checks covered
  1, 2, 10, 111, 112 and 400 frames, exact frame retrieval, bounded windows and
  worker shutdown. An isolated five-second character viewport check loaded once
  and performed no redraws after warm-up. This is an idle-loop check, not a
  playback FPS or whole-desktop performance guarantee.
- A 474-frame PMX animation was baked and reopened under a 295-character data
  path. All 215 animation channels retained exactly the same key values as the
  short-path baseline; no armature-object animation keys were added.
- Animation-only FBX and PMX VMD exports succeeded at 289-character destination
  paths. Blender reimport checks found finite sampled FBX poses and no object
  transform keys. Short- and long-path VMD outputs were byte-identical.
- Test scripts, character assets and generated outputs remained outside the
  public checkout. These checks did not repeat standalone MMD playback or test
  other GPUs, network shares or every possible Windows path.

## Original-resolution character textures (2026-10-27)

- Mixamo FBX, VRM and PMX samples retained their original image dimensions and UVs.
  A pre-existing PMX asset prepared its texture cache without reimport or retargeting.
  Sampled GPU-deformed vertex buffers were unchanged, including a 474-frame take.
- Isolated 1920 x 1080 GPU checks ran 600 draws per sample, excluding the first 50
  from timing statistics. Original-texture median / p95 draw times were
  0.108 / 0.264 ms (FBX), 0.331 / 0.536 ms (VRM), 0.466 / 0.697 ms (PMX rest),
  and 0.411 / 0.617 ms (PMX animation). The largest recorded draw was 2.50 ms.
  These are isolated renderer measurements, not complete application frame times.
- Twelve renderer load/close cycles kept the handle count constant. Process RAM
  rose about 13 MiB during warm-up and stabilized in the last three cycles.
  Three loader playback/seek cycles had no missing animation windows, constant
  handles and less than 0.1 MiB additional RAM. Timer lateness reached 15.7 ms;
  these checks do not establish video/audio synchronization or universal FPS.
  Cancelling background texture preparation shut the loader down in 0.107 seconds.
- Original images are cached once per character on CPU and uploaded once per
  loaded renderer, with mipmaps for distant viewing. Motion buffers remain bounded.
  Full PBR materials and general translucent-material sorting are not implemented.
  Tests and generated character assets remained outside the public checkout.

## Runtime memory-pressure policy (2026-10-28)

- Removed predicted RAM/VRAM admission failures from shared workers, FlowHMR and
  ExMR. FlowHMR's 5,000-frame limit, adaptive batches, FP32 temporal context,
  allocator reserves and the existing GVHMR full-key query-split path remain.
  Estimates can still select a computation path; they no longer reject it.
- A real 12-frame 1920 x 1080 input completed moving-camera VGGT, Fast SAM and
  FlowHMR generation while worker policies were given 24 GiB available RAM.
  This simulated the reported availability; the physical test PC had more RAM.
  VGGT peak process RAM was 8.89 GiB and peak Torch CUDA allocation 6.32 GiB.
  This does not establish a minimum RAM requirement or behavior on other GPUs.
- Qt timer/worker-tree integration tests simulated low available RAM, low Windows
  allocation headroom and critical pressure. Workers and their children stopped
  in 0.65–1.14 seconds; stage errors were saved and prior committed test data
  survived. One low sample followed by recovery did not stop a job. The largest
  observed 16 ms UI-timer gap was 31.8 ms in these short tests.
- Injected CPU MemoryError and CUDA allocation failures returned explicit failed
  events and retained existing motion files. A real FlowHMR retry completed and
  cleared its pending marker. Actual system RAM was never exhausted deliberately.
- Exact policy-only file-hash aliases preserve preceding generic/GVHMR signatures
  and all three ExMR stage identities. Later byte changes invalidate them normally.
  ExMR identity checks used synthetic input stamps and installed model statistics;
  full ExMR and GVHMR inference were not repeated. GVHMR allocation/retry control
  flow was exercised with a test model and simulated allocator observations.
- Tests and generated inputs/results remain outside the source checkout. Runtime
  monitoring cannot guarantee intervention before every sudden large allocation.

## ComfyUI-style external VRAM margin (2026-10-28)

- Shared, FlowHMR and ExMR CUDA allocator budgets now subtract 600 MiB from
  current free VRAM on Windows, or 700 MiB above 15 GiB GPU capacity, matching
  [ComfyUI's external-use margin](https://github.com/Comfy-Org/ComfyUI/blob/master/comfy/model_management.py).
  The old FlowHMR/ExMR fixed 3/2 GiB deductions and extra 95/90% caps are removed.
  ComfyUI's separate 0.8 GiB inference planning allowance is not another allocator
  deduction: weights, activations and temporary buffers share the allowed budget.
  This margin alone does not implement offloading or guarantee free memory against
  other applications or non-PyTorch allocations.
- Boundary checks covered simulated 4/8/12/15/16/24/32 GiB capacities, the exact
  15 GiB transition, exhausted free memory and all four allocator call sites.
  Existing batch backoff and GVHMR dense-OOM/full-key retry checks passed.
- On the RTX 5090 test PC, all three isolated runtime profiles applied a 700 MiB
  margin, performed CUDA allocations, raised a real allocator OOM under a tiny
  test cap and successfully allocated again after restoring the normal budget.
  The test deliberately limited the allocator rather than exhausting the GPU.
- A private 12-frame 1080 x 1920 clip completed FlowHMR camera/Fast SAM/generation,
  GVHMR observations/generation and ExMR keypoints/features/generation using the
  installed model weights. Other GPUs and long clips were not tested here.
  FlowHMR generation from identical cached features produced identical motion
  tensors with the old and new allocator caps.
- Existing generic/GVHMR cache fingerprints and all three ExMR stage identities
  were retained. Test code, inputs, generated motion and logs remain outside the
  public checkout under the private validation artifacts directory.

## Memory-policy follow-up review (2026-10-28)

- Fixed GVHMR's dense-OOM retry report to compute `fits` from observed capacity
  rather than always reporting a shortfall. The retry still preserves FP32 and
  the full key/temporal context and does not reject work using that estimate.
- Made worker cancellation idempotent while its Windows termination helper is
  active. Late helper callbacks check that the worker still exists and only clear
  their own helper reference. Four callback/worker-lifetime combinations, each
  with five duplicate requests, passed without Qt callback exceptions.
- ExMR now replaces a previous failed stage status after successful computation
  or validated cache reuse. Its explicit force flag also bypasses the cache.
  All three cached ExMR stages repaired failed status without altering tensors;
  a forced 12-frame generation was separately checked for a non-cached completion.
- Each model's actual worker entry reported injected CPU allocation failures and
  real CUDA allocator OOMs with an error, exit code 1 and unchanged prior motion.
  Tiny allocator limits triggered CUDA failures without exhausting system VRAM.
  Subsequent retries completed using existing preprocessing data.
- Four Qt timer/worker-tree tests simulated low RAM, low Windows allocation
  headroom and critical pressure. Stops completed in 0.66-1.17 seconds, preserved
  committed results and left no test child worker running. The largest observed
  16 ms UI heartbeat gap was 29.12 ms; a transient low sample did not stop work.
- Exact RAM/VRAM boundaries, Windows commit observations, traceback cleanup,
  exhausted-batch errors, existing inference cache identities and source checks
  passed. This is a single-PC regression check, not a low-memory hardware test or
  a guarantee against sudden allocations or competing GPU applications.

MAMMA multi-view input setup (2026-10-29):

- Five real camera videos opened through the multi-select picker and produced a
  four-column thumbnail grid with a second row. Ten preview-camera switches kept
  the frame, trim, 3D camera and one persistent decoder; sampled switch latency
  was approximately 0.03–0.21 seconds on cached local files. The short sequence
  showed no net thread/handle increase; this is not a long-run leak guarantee.
- Real 25 fps / 60 fps videos shared a 30 fps timeline limited by the shorter
  input. At timeline frame 45 (zero-based, 1.5 seconds), native requests were 38
  and 90. Playback and resume after switching were exercised in the Qt event
  loop; this was not a visible full-app GPU FPS measurement or a VFR test.
- The actual Save Project As worker copied every input, restored common trim,
  active camera, manual calibration and correction selection, and saved a second
  portable project copy. Reopening in another tool and returning to MAMMA also
  recovered the saved inputs. Eight rapid replacement requests coalesced to the
  latest set, and saving was blocked during input preparation.
- JSON and MAMMA YAML camera imports were checked, including camera-to-world
  quaternion conversion against matching world-to-camera matrices. Invalid
  rotations/intrinsics and missing files retained previous valid inputs/settings.
  Existing three-model controls and single-video decoding were checked.
- These initial checks covered input setup only. Later native-pipeline checks
  below extend their scope; they do not validate fresh-user installation.
  Original videos and motion data were not modified. Private test files remain
  outside the public checkout.

MAMMA native app pipeline (2026-10-29, same tested PC):

- The GUI Reconstruct 3D queue completed SAM 2.1 segmentation, exact-ROI FP32
  landmarks, VGGT cameras, full FP32 MAMMA fitting, native actor conversion,
  shared-floor correction and source export preparation. Two synchronized views,
  two people and 30 frames (one second) took 299.50 seconds including worker
  startup and native preparation. A verified repeat reused inference in 14.48
  seconds and retained the raw fitted file's timestamp and size.
- Recorded PyTorch peak allocated/reserved VRAM (GiB) was 2.532/2.846 for masks,
  0.585/0.645 for landmarks, 6.097/6.121 for cameras and 0.315/0.352 for fitting.
  The largest process peak working set was 8.928 GiB in camera estimation.
  These are process/allocator observations, not whole-PC or driver VRAM totals,
  minimum requirements, or measurements for a long clip.
- Exact sixteen-beta conversion matched original fitted meshes within 0.001 mm.
  Original and corrected previews for two different body templates rendered
  together on the real GPU. Readback against the CPU skin reference at three
  frames differed by at most 0.002 mm. Twenty-four visibility changes retained
  constant sampled handle/thread/GPU-buffer counts; sampled RSS changed about
  0.04 MiB after warm-up. This is not a long-run leak guarantee.
- Camera switching and six original/corrected transitions preserved the selected
  frame and 3D camera. In a three-second playback check, the GUI event-loop gap
  p95 was 9.57 ms and maximum 15.64 ms. These are event-loop gaps, not rendered
  frame times or a universal FPS claim. Original GPU images were inspected.
- Save Project As/reopen retained both actors and both working camera previews
  with the original project temporarily offline. Manual lock edits blocked stale
  export; Apply Correction refreshed both actors in 15.85 seconds without
  changing either original source tensor. Incomplete publication, changed input
  receipts and invalid saved settings were rejected. Removing a required cached
  landmark rejected the cache and allocated a separate attempt. Missing
  project-private body data was regenerated from the unchanged raw fit in
  16.31 seconds without rerunning inference.
- Four real source FBX exports (two people, original/corrected) imported through
  ordinary Blender 4.5 settings with 52 Hips-root bones and no armature-object
  keys. Maximum imported bone-head difference was 0.0034 mm. Retargeted-character
  output and manual-camera end-to-end inference were not tested in this pass.
- Model Manager imports were registered locally with expected SHA-256 receipts.
  No registered weights, external MAMMA engine, body-derived assets, videos or
  private tests were added to the publishable checkout. See [MAMMA](MAMMA.md)
  for its separate-environment limitation; this is not a clean-install or
  other-GPU validation, nor a validation of the 60-second adapter limit.

Managed CMD startup after MAMMA routing changes (2026-10-29):

- Reviewed unchanged dependency pins and refreshed the installer lock's registry
  hash. The standard managed installation worker verified and reused all six
  installed profiles without runtime downloads or replacement. It refreshed the
  installed verification state; startup checks were not bypassed.
- Actual RGBMoCap.cmd startup and its child-worker environment check passed.
  A MAMMA project then opened through that same launcher and played two people
  for three seconds without render errors or observed video/mesh frame mismatch.
- Source composition validation now rejects a runtime registry hash that differs
  from the installer lock. A private fixture confirmed both valid acceptance and
  mismatched-lock rejection; private files remain outside the checkout.

These checks describe the recorded examples, not a guarantee for every video,
character or computer. Model weights and registered body data remain separate and
subject to their providers' terms. Review the applicable third-party notices and
corresponding source/build requirements before redistributing binaries.

MAMMA supported optional runtime and UI (2026-10-29, same Windows/RTX 5090 PC):

- A new isolated Python 3.11.17 / torch 2.7.1+cu128 runtime was installed from
  SHA-256 pinned package/source archives, without copying the research venv.
  SAM 2 and SDF were compiled locally. 176 package versions and 41,138 runtime
  files were checked; deliberately pruned debug/developer files were excluded.
  Actual runtime size: 6.19 GiB, excluding model weights and compiler tools.
  Re-running setup reused the verified runtime without downloading/rebuilding.
- CUDA SAM connected components and SDF distance, index width, winding and point
  gradients passed. SDF squared-distance max error: 1.21e-8; gradient error: 0.
  Restricted upstream code/native binaries remain outside this repository.
- Fresh two-view, two-person, 30-frame processing completed in 297.55 s through
  correction/preview. Peak PyTorch allocated/reserved memory was 6.10/6.12 GiB;
  Windows process peak working set was 8.93 GiB. These are different measures
  from whole-board VRAM or total system use. On the same working inputs, all
  compared landmarks, visibility/contact predictions, fitted poses, betas and
  translations matched the previous research engine exactly (maximum 0).
- Original native vertices matched fitted MAMMA vertices within 0.001 mm.
  Four source FBX files imported in Blender 4.5 with 52 Hips-root bones and no
  armature-object animation; max sampled bone-head error was 0.0034 mm.
- Height is numeric-only with a separate metre label; People has a separate
  label, and MAMMA hides Find person. Nine model assets, runtime readiness/setup
  instructions, other models' button restoration and the model guide were checked.
- Fixed asynchronous thumbnail restoration capturing a temporary default 3D
  camera. Fresh projects without a saved view framed both actors; saved camera
  target/distance were preserved on reopen. One earlier direct-GUI reopen timed
  out; subsequent fresh/saved-camera checks and CMD playback passed. This does
  not establish long-duration reliability or performance on every project.
- Actual RGBMoCap.cmd playback (4 s): 29.74 fps, frame interval p95 34.78 ms,
  max 35.65 ms, video/3D and mesh/video frame differences 0; two actors and name
  labels were visibly present, render errors absent, paused skin dispatches 0.
- Setup failure injection retained the old pointer and model/project sentinels
  and removed its failed staging directory. Unsafe archive aliases were rejected;
  generated batch paths with Korean text, percent signs and shell punctuation
  were checked. This is not a full non-ASCII compiler-install test.
- User's separate manual three-view/413-frame log completed in 712.08 s without
  fatal traceback or OOM. No clean-Windows, other-GPU, 60-second/eight-person
  capacity, or newly retargeted-character test is claimed in this pass.

MAMMA camera-count restriction removal (2026-10-30, same Windows PC):

- Removed the GUI/worker four-view maximum and the input/calibration parser's
  separate 64-camera ceiling. Reconstruction still requires at least two views;
  duplicate names, calibration/size checks and bounded metadata files remain.
  Existing clip/person limits were not changed.
- Input, calibration, request and saved-media validation accepted 2, 4, 5, 8, 16,
  32 and 65 camera entries. Empty/duplicate inputs, a single-view reconstruction
  and mismatched camera calibration were rejected.
- The actual preparation worker decoded/resampled all 5, 16 and 65 test videos,
  retaining 12 frames per view. Latest observed preparation times were 1.35,
  2.93 and 9.88 seconds. These were tiny synthetic 320x180 clips with repeated
  content for input-count regression, not multi-camera reconstruction benchmarks.
- Actual GUI widgets loaded 5, 16 and 65 thumbnails, selected the final camera
  and retained the shared trim and timeline frame. The 65-view GUI request queued
  all six reconstruction stages. Save Project As and actual GUI reopening
  retained all 65 media references, calibration and the selected last camera.
  No 3D render/playback performance claim is made from this hidden-window test.
- Shared system-RAM/Windows allocation-headroom checks and CUDA allocator budgets
  were unchanged. Low-memory observations triggered the existing guard. Injected
  CUDA OutOfMemoryError and CPU MemoryError produced failed events and nonzero
  worker results while preserving the existing project manifest. This was fault
  injection, not deliberate exhaustion of system/GPU memory.
- No new 5/16/32/65-view SAM, VGGT or complete MAMMA fitting run was performed.
  Removing a fixed input-count ceiling is not a measured memory-capacity guarantee.
  Private test scripts, synthetic media and logs remain outside the source checkout.

Pre-publication regression audit (2026-10-30, same Windows 11 / RTX 5090 PC):

- A separate source/runtime copy and local-only Git remote exercised the
  pre-MAMMA-to-current update, `install.cmd` verification/reuse and actual
  `RGBMoCap.cmd` startup before the optional MAMMA runtime was registered.
  Model/project/cache sentinels survived. Tracked local edits blocked updating;
  the application lock blocked a second launch. This reused installed pinned
  runtime files; it was not a clean-Windows or fresh-download installation.
- `Setup_MAMMA.cmd` accepted the license response, performed full package/native
  CUDA verification and retained the verified runtime. Declining left it intact.
  Injected setup failure retained the old pointer and user-data sentinels;
  archive escapes were rejected. Fresh native compilation was not repeated in
  this audit; its earlier build evidence remains recorded above.
- Fresh ExMR, FlowHMR and GVHMR processing each used the same 60-frame real-video
  segment. FlowHMR included VGGT and Fast SAM; GVHMR included HTD, FootMR and
  HandFlow. Reconstruction, correction, preview preparation and source FBX
  saves completed. MAMMA separately processed two synchronized views, 30 frames
  and two people through new SAM 2.1 masks, landmarks, VGGT, fitting and publication.
- Seven source FBX files were actually imported in Blender 4.5. All retained
  52 Hips-root bones and bone-only animation. Maximum sampled bone-head error
  was below 0.008 mm. Character imports covered Mixamo FBX, VRM and a 797-bone
  PMX; all four source models retargeted successfully. Eight character FBX
  reimports covered mesh/motion-only options, original rest transforms and
  sampled poses, with no armature-object or hair/clothing animation channels.
  PMX VMD exports passed finite-channel/frame/structure checks; native MMD
  playback was not repeated in this audit.
- Fixed MAMMA preparation/publication failures incorrectly returning process
  exit code zero. Both now emit failure and exit nonzero. Fixed retargeting's
  MAMMA body selection; a real PMX retarget also passed while the separate
  standard SMPL-X file was unavailable in the private test copy.
- Real GUI checks covered all four scoped model managers, original/corrected
  switching without frame/camera changes, manual contact editing, stale export
  blocking, Apply Correction without inference, Save Project As/reopen and
  MAMMA camera switching/private-body relocation. Eight project replacements
  were sampled; this short test does not prove leak freedom. Cached switching
  among three models in one project, GVHMR result variants, GUI character
  import/retarget, linked-character refresh after correction and selected-row
  character removal also completed.
- Four-second actual CMD playback samples for each model measured 29.74 fps,
  frame-interval p95 34.64–35.21 ms and maximum 35.28–36.74 ms. Video/3D and
  mesh/video frame differences were zero; paused skin dispatches were zero.
  These short clips on one PC are not long-video or cross-GPU guarantees.
- A separate 797-bone PMX viewport check with two MAMMA source actors measured
  29.73 fps over four seconds (p95 36.44 ms, maximum 37.87 ms), with no character
  renderer error. Two-second rest-pose and paused-animation observations each
  recorded zero loader-change events and zero frame swaps. This specifically
  checks the short/final-window repaint regression, not long-duration stability.
- Test scripts, licensed inputs, runtime copies, logs and generated motion were
  kept outside the publishable checkout. Main, user, per-model, build and agent
  guides were checked against current controls, file requirements and limits.

### Person-selection responsiveness (2026-10-30)

A two-person MAMMA project's independent copy was switched six times before
and after the GUI change. With profiling disabled, median synchronous selection
time fell from 803 ms to 173 ms; selected-preview/video readiness fell from
1,117 ms to 534 ms. The common scene, source frame and world camera were retained.
An unchanged active camera was reopened zero times instead of twice per switch.
The remaining selected-person preview loading still runs in the background.

Nested control updates now validate readiness once at their synchronous outer
boundary. Shared-floor checks reuse one read-only snapshot inside an individual
GUI validation call, never across calls or threads. Export and later callbacks
still check disk afresh. Tests confirmed that pending publication, changed floor
receipts and unapplied manual contact edits block export after a preceding valid
check. Manual contact editing, actor switching and Apply Correction completed
without replacing inference. The live non-default camera choice was retained.
Existing ExMR, FlowHMR and GVHMR cached original/corrected views also rendered.
Video reuse checks file identity, FPS and timeline offset; changed inputs and
decode failures reopen normally. These are short same-PC measurements, not
long-recording or other-hardware guarantees. Test data remains outside source.

Four-second CMD playback samples across the four models measured 29.74–29.82 fps,
with frame-interval p95 34.71–35.04 ms and maxima 35.86–36.73 ms. Video/3D and
mesh/video frame differences were zero. GVHMR finished one queued frame just
after Stop; a separate settled two-second pause recorded zero skin dispatches.

### Automatic / Low Memory routing (2026-10-30)

Fresh independent runs on the tested Windows/RTX 5090 PC covered both modes:
ExMR and FlowHMR each processed 60 frames through correction and export; GVHMR
also ran HTD, FootMR and HandFlow. MAMMA processed two synchronized views,
two people and 30 frames through fresh masks, landmarks, VGGT, fitting and
correction. Existing separate pinned runtimes were reused, not reinstalled.
FlowHMR's Original SAM (Fast SAM off) also completed both modes on an independent
eight-frame feature-extraction fixture.

Automatic selected fast paths with spare GPU memory. Allocator-budget tests
selected Low Memory and completed ExMR preprocessing at 3.9 GiB, body/hand
features at 4.5 GiB, FlowHMR camera at 3.5 GiB, and SAM features/motion at 3 GiB.
A real fast-loading CUDA OOM was released and retried once successfully in
Low Memory. ExMR features at 4.1 GiB still failed safely with memory advice;
the batch remained 8. These are caps on a 5090, not validation of real 3–4 GiB
GPUs. Models and full-sequence context can still exceed available memory.

Automatic versus Low Memory body-joint differences in the three single-video
samples were at most 0.84 mm before correction and 0.53 mm after correction;
separate inference runs are not bit-identical. MAMMA camera matrices and both
actors' sampled exported joints/rotations were identical in this comparison.
Ten corrected-source FBX files imported in Blender 4.5 with 52 Hips-root bones,
bone-only animation and sampled positions within 0.2 mm of committed frames.
Seven policy/retry/hook-lifecycle checks and actual Console/CMD mirroring passed;
the saved Automatic selection was retained. A short hook test does not prove
long-run leak freedom. Playback/rendering code was unchanged in this revision.

ExMR Sapiens peak PyTorch allocation decreased from 5.68 to 3.10 GiB; combined
body/hand feature-stage peak decreased from 5.15 to 3.87 GiB. These exclude
driver/display allocations. Low Memory can increase elapsed time and host RAM;
single-pass cold-start timings are not speed benchmarks. Precision, frame count,
resolution, temporal context and fitting iterations were retained. RAM/Windows
allocation-headroom monitoring remains a stop condition. Test scripts, weights,
logs, motion and temporary fixtures remain outside the publishable checkout.

### FlowHMR Fast SAM acceleration controls (2026-10-30)

The app now saves two independent, default-on switches for bounded four-frame
batching and GPU hand crops. Four combinations completed a real 60-frame input
with 30 valid pelvis anchors each. Against both switches off, final joint-position
differences were 0.21 mm at the 95th percentile / 1.52 mm maximum for batching,
and 1.23 mm / 2.45 mm with both switches on. These measure output disagreement,
not ground-truth accuracy. The separate earlier short-clip speed tests remain
the basis of the tooltip estimates; this integration check is not a new paired
speed benchmark.

Peak PyTorch allocation was 3.44 GiB without batching and 3.84 GiB with batching.
A seven-frame Low Memory run used one-frame batches with GPU crops and peaked
at 1.92 GiB. Original SAM completed three frames with acceleration flags ignored.
One-, two- and three-frame partial batches completed with finite outputs. CUDA
calls are not claimed bit-identical. Precision, crop resolution and the 20-step
FlowHMR generator were retained. No new packages or runtime installation were
needed; no forced physical OOM test was performed in this integration check.

Qt checks covered defaults, all four saved option combinations, busy-state
disablement, Fast SAM off, and all four model pages at short/tall window heights.
The settings use consistent top-down spacing and scroll when needed; hidden
model pages no longer reserve the tallest page's height. Test files and logs
remain outside the app checkout.


## Sequential multi-person model reuse (2026-10-30)

The app now groups ExMR, FlowHMR and GVHMR actor/take jobs by supported stage.
One model serves actors sequentially, with explicit actor paths and separate
receipts. GVHMR runs its observation and core phases in one worker, closing each
model scope before the next phase. Optional HTD, FootMR and HandFlow keep their
existing separate workers. Single-actor jobs keep their previous queue.

App-path measurements on the documented Windows RTX 5090 PC used two tracked
people with 60 and 45 frames at 30 fps and static cameras. Timings include feature
and core worker startup, excluding discovery, camera estimation, correction,
export and optional add-ons. These are one-trial observations, not universal
speedups; import and filesystem warm-up vary.

| Model | Separate actor workers | Stage reuse | Peak PyTorch allocated VRAM, before → after |
| --- | ---: | ---: | ---: |
| FlowHMR | 108.69 s | 65.34 s | 3.84 → 3.84 GiB |
| ExMR | 161.08 s | 106.51 s | 5.68 → 5.68 GiB |
| GVHMR | 56.02 s | 22.97 s | 2.82 → 2.82 GiB |

All six fixed-input actor/model comparisons were tensor-exact. With fresh image
preprocessing, maximum joint differences against the preceding experiment were
0.344 mm for FlowHMR, 0.242 mm for ExMR and zero for GVHMR; upstream preprocessing
roundoff is not a claim of improved or degraded accuracy.

Sixteen unit checks covered model lifetime, replacement, error cleanup, allocator
budgeting, OOM retry, queue ordering, bounded requests and path/runtime validation.
Real workers checked cache hits without model construction, two-frame Low Memory
feature extraction for FlowHMR/ExMR, Original SAM, corrupt ExMR hand-feature
recovery, and correction/derived export for all three models. The app writer also
saved three binary FBX files; Blender reimport was not repeated for this change.
A controlled real CUDA allocator OOM in a separate 1%-capped test process recovered
once in Low Memory and ended with zero allocated tensor bytes. This is not a
full-model OOM stress test on another GPU.

Actual GUI QProcess runs checked repeated cached queues and cancellation during
actor B, followed by reuse of completed actor A and reconstruction of B. A 20 ms
GUI timer had a recorded worst delay of 45 ms during that cancellation/retry test;
this measures offscreen event-loop responsiveness, not viewport FPS. Repeat checks
were short and do not establish universal leak freedom. GVHMR retained a stable
32 MiB PyTorch allocation between phases and its worker then exited. No new
runtime packages or changes to the pinned runtime registry were needed. Private
fixtures, model links and test scripts remain outside the application checkout.

## Pipeline and CPU preparation follow-up (2026-10-30)

A separate Windows RTX 5090 experiment retained the pinned runtimes, native
precision, batch sizes, resolution and temporal context. Two trials reversed
execution order; stages released their model before the next stage loaded.
AI worker times include startup and saved results but exclude discovery,
camera estimation, correction and export.

| Model / people | Separate phase workers, two trials | Connected phases, two trials | Peak allocated VRAM |
| --- | ---: | ---: | ---: |
| FlowHMR / 1 | 58.65, 53.33 s | 41.12, 44.18 s | 3.84 GiB in both |
| ExMR / 1 | 84.83, 82.88 s | 46.32, 46.99 s | 5.68 GiB in both |
| ExMR / 2 | 94.90, 90.76 s | 52.78, 53.81 s | 5.68 GiB in both |

Inputs were 60 frames for one actor and 60+45 frames for two actors, at 30 fps
and 720x1280. FlowHMR's two-person connected-phase benefit was smaller than
trial variability, so its existing multi-person phase grouping was retained.
Short-clip startup gains do not predict the same percentage on long footage.

Warm per-stage tests used those clips plus a separate 120-frame 1080x1920 clip,
each processed twice: 450 frames per variant. CPU preparation ahead by at most
two batches reduced total measured Sapiens time from 11.53 to 8.63 s, ExMR body
features from 18.57 to 13.75 s, and hand features from 7.50 to 5.63 s. Flow Fast
SAM changed from 42.96 to 40.79 s. Peak CUDA allocation was unchanged and sampled
RAM growth was small. ExMR final body-readout omission alone took 16.63 s; with
preparation overlap it took 13.15 s. The analogous hand-readout omission had no
clear benefit and was not adopted. These stage gains cannot be added together
to claim a whole-job speedup.

Sapiens and hand features were tensor-exact; ExMR body features differed by up
to 4.8e-6, versus 3.9e-6 on baseline repeats. Optimized final motion differed by
at most 0.83 mm / 0.306 degrees for FlowHMR and 0.88 mm / 0.068 degrees for ExMR.
Baseline repeats also differed, by up to 0.77 mm / 0.217 degrees and 0.69 mm /
0.047 degrees respectively. This is an output-preservation check without ground
truth, not a claim of improved or degraded tracking accuracy. Fixed feature
inputs produced tensor-exact motion when phases were connected.

Automatic CPU preparation is limited to one producer and two prepared batches,
with an additional 256 MiB prepared-storage estimate. It is disabled in Low
Memory or with insufficient CPU/RAM headroom. Existing OOM recovery and actual
system-memory monitoring remain active. No new runtime packages were added.

The integrated app code passed 25 focused policy/lifetime checks and five
tensor-exact grouped-versus-independent motion comparisons on fixed features.
Real one-/two-person worker runs verified model release and cached reruns without
loading a model. Offscreen GUI QProcess checks covered automatic single-person
phase joining, cancellation in the second actor's body extraction and retry
using completed preprocessing. A 20 ms GUI timer had a maximum observed delay
of 44.92 ms during that cancellation/retry run; this is not a viewport FPS test.
Eight-frame real clips checked Low Memory for FlowHMR/ExMR and FlowHMR Original
SAM. A damaged private ExMR hand-feature cache recovered without recomputing
valid body features. Correction and binary FBX saving were checked for both
models; Blender reimport was not repeated for this optimization.

## Contact correction v9: coherent root and swing transitions (2026-10-30)

Cached-source regression covered ten sequences / 10,121 actor frames: two
MAMMA actors, ExMR slow motion and stairs, FlowHMR long motion and backflip,
GVHMR dance, and pushups from all three single-video models. No AI inference
was rerun. Contact timings and stance ankle targets were retained. This is a
correction comparison without motion-capture ground truth.

The planner now joins swing targets to the final root-shifted free trajectory,
fades its extra target adjustment at the free boundary, and evaluates a smooth
floor-clearance constraint inside the root search. Native anatomical IK still
publishes leg rotations once. Existing 1,024-frame cores, four-second halos,
2 cm planning budgets, 60-evaluation cap, source ankle frames and hand solver
remain. Jacobian column scaling improves numerical conditioning. An exhausted
planning window is reported explicitly; it is not a convergence claim.

For the first MAMMA actor, pooled knee short-period RMS fell from 0.935 to
0.332 degrees (64.5%); the second fell from 0.390 to 0.329 degrees (15.6%).
The identified free-leg join's added bend step fell from +14.22 to -0.41
degrees. Short-period RMS subtracts a nine-frame degree-two Savitzky-Golay
trace; it is a diagnostic, not ground-truth error. Source-relative knee
second-difference RMS decreased on all ten sequences. Individual metrics do
not all improve: backflip total short-period RMS increased by about 2%, and
the stairs' worst added step increased by about 0.23 degrees. Residual source
noise, incompatible/unreachable pins and rapid intended movement remain.

MAMMA full-contact skinned sole height error stayed within 1.36 mm; across the
other tested foot contacts it stayed within 1.14 mm. Stance ankle positions
changed by less than 0.001 mm, fixed lengths and source ankle orientation were
retained, and torso world rotations were preserved. Hand timings and arm-only
behavior were checked; existing unreachable-palm residuals remain explicitly
reported. The backflip retained 103 contact-free flight poses exactly. Manual
intervals, disabled feet, empty timings, one-frame bypass and native window
sizes 64 versus 1,024 were checked. Two-, three- and seven-frame contact clips
remained finite. An intentionally exhausted planning cap produced its warning.

CPU correction-only measurements, excluding body sampling, hands, inference,
rendering and export: the MAMMA pair changed from 1.42 to 1.07 s (sum of three-run
medians); the long FlowHMR case increased from 1.34 to 1.53 s. Other long cases
were faster in these runs. Sampled process peak working set stayed below
300 MiB for these numeric-input tests; this is not whole-app RAM usage. No GPU
allocation or additional dependency is introduced by this change.

A separate copied MAMMA project completed the installed correction, preview
and binary FBX workers. Ordinary Blender 4.5 import checked all 422 frames,
52 bones, rest transforms, terminal tips and sampled half frames: maximum
joint error 0.0044 mm and fractional rotation error 0.0028 degrees against the
committed export. This was not a new playback-cadence or clean-install test.
User projects were not recalculated or replaced; Apply Correction is required
to obtain the new result. Private fixtures and media are outside the checkout.

## Final pre-publication review (2026-10-30)

Rechecked the 25 queue/prefetch/model-lifetime tests, four-model settings layouts,
Fast SAM option persistence and a real GUI connected-worker cache rerun. Current
correction code also passed the short-clip, contact/flight and installed-versus-
isolated checks above. Two-actor application-worker smoke tests completed for
FlowHMR, ExMR and GVHMR using private projects; these included fresh ExMR/GVHMR
processing and a FlowHMR cache hit. GVHMR retained 32 MiB of cuBLAS workspace
after releasing its model: a separate diagnostic cleared that library workspace
and measured zero remaining allocation. This is not a model leak or a guarantee
of zero CUDA context memory. No new runtime packages or clean-PC tests were added.
Source syntax, manifest hashes, local documentation links and publishable file
composition were checked; private tests, assets and logs remain outside the repo.

## Automatic VGGT meta/assign loading (2026-10-30)

Only the camera-only fast loader now uses meta initialization and direct
checkpoint assignment. Low Memory and its Automatic retry retain copy loading
and the existing block offloader; generic depth loading keeps its previous path.
No packages, model precision, sampled frames or camera equations changed.

The preceding independent component study ran 14 configurations across the Flow
and MAMMA Python runtimes. Camera pose encodings, intrinsics and extrinsics were
exact; six real load/infer/close cycles and one controlled failure cleared model
references and hooks. See [the measured scope](PERFORMANCE.md#vggt-camera-checkpoint-loading).

App integration used copied private data, not user projects. FlowHMR's actual
camera worker processed a 60-frame moving-camera clip with old/new fast loading,
Low Memory and one deliberately injected CUDA transfer OOM. All numerical camera
arrays were exact. The OOM test observed meta/assign on the first attempt, disposal
of that model, then non-meta copy loading on the Low Memory retry. MAMMA's separate
runtime processed its camera/capture worker from cached masks/landmarks for a
30-frame, two-view clip. Old fast, new fast and Low Memory produced exact world
cameras, estimated scale and calibration YAML. No new mask/landmark/body fitting,
full-motion quality, Blender or clean-PC test is claimed for this loader change.

Those seven camera-worker runs passed. Two additional checks confirmed a FlowHMR
camera cache hit loads no VGGT model and Static Camera still skips VGGT with zero
recorded GPU allocation. Five focused loader checks covered fast/Low Memory
selection, unchanged generic depth routing, missing checkpoint rejection and
unmaterialized-meta rejection. Test files, raw logs and assets stay outside the repo.
