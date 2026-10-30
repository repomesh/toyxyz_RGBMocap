# ExMR

[Original project](https://github.com/twehrbein/ExMR) · [All model guides](README.md)

ExMR reconstructs SMPL-X body and finger motion from one video. It is the default
model for new work. In this project's subjective tests it was generally fast
with good motion quality; results depend on the footage and settings.

## Prepare files

Run the normal `install.cmd`; no separate ExMR setup command is needed.
Select **ExMR > Model Manager** and prepare the listed files. Key assets are:

| Asset | Location under the app's `models/` folder |
| --- | --- |
| ExMR weights | `exmr/exmr.safetensors` |
| Sapiens2 pose weights | `exmr/sapiens2_1b_pose.safetensors` |
| SMPL-X v1.1 neutral body | `body_models/smplx/SMPLX_NEUTRAL.npz` |
| MANO right hand | `body_models/mano/MANO_RIGHT.pkl` |
| Shared SAM 3D Body model, config and MHR | `flowhmr/sam/model.ckpt`, `flowhmr/sam/model_config.yaml`, `flowhmr/sam/assets/mhr_model.pt` |
| HaMeR and hand mean | `handflow/hamer.ckpt`, `handflow/mano_mean_params.npz` |

Model Manager also lists shared tracking dependencies. Use its exact supported
files and import checks; do not substitute arbitrary checkpoint versions.
Registered SMPL-X/MANO and gated SAM 3D Body files may require manual download.
The shared `flowhmr` and `handflow` folder names do not mean you must run those
models or enable GVHMR's HandFlow add-on. ExMR uses these assets directly.
MAMMA's locked-head SMPL-X bodies are different files and cannot replace ExMR's
standard body. See [model download and reuse](../USER_GUIDE.md#prepare-models).

## Controls

| Control | Default for new work | Meaning |
| --- | --- | --- |
| **Camera** | Static Camera | Use for a camera that stays still without zooming. Choose Moving Camera / SimpleVO when it moves. ExMR does not use FlowHMR's VGGT camera stage. |
| **Smooth Motion** | On | Reduce small frame-to-frame changes; off preserves the unsmoothed result. |
| **Resource Mode** | Automatic | Use fast loading when GPU memory is sufficient; otherwise use CPU-first body/hand loading and prepare Sapiens' existing FP16 layers before GPU transfer. Low Memory forces these paths. Body/hand batches remain 8. CPU loading can be slower and use more RAM. |

Camera and smoothing changes take effect on **Reconstruct 3D**, not on
**Apply Correction**. Existing projects retain saved settings.

## Capture and export

1. Select ExMR and use **File > Open Video**. Trim the **Video** track as needed.
2. For explicit identity selection or several people, run **Find person** and
   check who to process. The selected row controls preview and contact editing.
3. Prepare models, choose camera settings and run **Reconstruct 3D**.
4. Review the original and corrected motion. Edit foot/palm support intervals
   and **Apply Correction** when needed. This reuses inference results.
5. Use **File > Export FBX** for the displayed applied source motion, or
   [import and retarget a character](../USER_GUIDE.md#character-retargeting).

Processing uses a 30 fps working clip and preserves the source video. Saved
results are under the project's `exmr/` folder. Switching models loads that
model's own available results; it does not convert one model's inference into
another's. **Save Project As** copies saved work to the selected location.

## Limitations and resources

Reconstruction reuses one model at a time across selected people and connects
its processing phases in one worker. With enough CPU/RAM headroom, Automatic
prepares up to two image batches ahead; Low Memory disables this overlap.
The body feature extractor omits unused final pose readouts while retaining
intermediate feedback. Precision, image resolution and motion context are unchanged.

### Measured effects

Measurements below used an RTX 5090 32 GB, Ryzen 9 9950X and 96 GB RAM on
Windows. They use different baselines and must not be added together:

| Comparison | Measured change |
| --- | --- |
| Automatic → Low Memory, 60 frames including correction/preview preparation | Allocated VRAM **5.68 → 3.87 GiB** (32% less); RAM working set **11.88 → 11.94 GiB**; **89.01 → 94.78 s**. This mode check predates the latest acceleration. |
| Connected phases, one person / 60 frames | Worker time **83.86 → 46.66 s** (44% less); peak allocated VRAM **5.68 GiB** in both. Includes worker startup/saving, not discovery, correction or export. |
| Connected phases, two people / 60+45 frames | **92.83 → 53.29 s** (43% less); peak allocated VRAM unchanged at **5.68 GiB**. |
| CPU preparation overlap, warm feature stages | About **25% less time** for Sapiens and hand features; **29% less** for body features with unused final readouts omitted. Measured VRAM was unchanged; RAM was similar, with small increases in some stages. |

CPU-first loading can increase RAM substantially: the isolated hand-model test
rose from **1.54 to 5.79 GiB** working set. It is not a host-memory-saving feature.
Fixed-feature motion comparisons were exact; fresh optimized preprocessing
produced maximum joint differences of **0.88 mm**, with baseline repeats also
differing by up to 0.69 mm. These are output differences, not ground-truth accuracy.
See [full conditions, loading trade-offs and measurements](../PERFORMANCE.md).

Occlusion, motion blur, poor visibility and identity changes can produce wrong
poses or fingers. Automatic contact detection can also miss intentional support;
edit its timings rather than expecting correction to recover missing evidence.
Contact correction is distinct from Smooth Motion and preserves original results.

RAM/Windows allocation-headroom monitoring and the shared CUDA allocator budget
remain active. Allocation failure stops the job with an error; shorten footage
or free resources if necessary. Historical [processing/VRAM measurements](../USER_GUIDE.md#recorded-processing-time-and-vram)
are individual runs, not minimum specifications. See [validation](../VALIDATION.md)
for the tested PC and scope, and [licenses](../../THIRD_PARTY_NOTICES.md) for
upstream restrictions.
