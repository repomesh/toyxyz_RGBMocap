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
  This does not implement ComfyUI's offloading or guarantee free memory against
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
