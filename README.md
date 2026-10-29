# RGB MoCap

![RGB MoCap workspace showing video tracking and a 3D motion preview](docs/images/rgbmocap-workspace.png)

A native Windows app that turns video into editable human animation. Review and
export motion with AI processing running locally.

## Sample videos

### Motion capture

https://github.com/user-attachments/assets/b948dd8c-43b9-493a-9d7f-2fae0317695e

[Download original sample](docs/samples/motion-capture.mp4)

### Character retargeting

https://github.com/user-attachments/assets/1759f1bf-a0d0-409f-84d8-dde9409cfe6f

[Download original sample](docs/samples/character-retargeting.mp4)

## Install and run

1. Install [Git for Windows](https://git-scm.com/download/win). Open **Command Prompt**
   in a writable folder where you want to install RGB MoCap, then clone the repository:

   ```cmd
   git clone https://github.com/toyxyz/toyxyz_RGBMocap.git
   cd toyxyz_RGBMocap
   ```

2. Prepare the runtimes for ExMR, FlowHMR and GVHMR with their supported add-ons,
   Blender, FFmpeg and native motion correction. AI weights and registered body
   files are downloaded separately in Model Manager:

   ```cmd
   install.cmd
   ```

3. Start the app:

   ```cmd
   RGBMoCap.cmd
   ```

   Keep the console open while the app runs. Prepare the AI model files separately
   using **Model Manager**; some require account registration and manual download.

To update, close the app and run `update.cmd` from the same folder. See the
[installation guide](docs/USER_GUIDE.md#install) for disk space, prerequisites
and recovery.

### Additional setup for MAMMA

For multi-view capture from synchronized camera videos, complete the base
installation above, then install **Visual Studio 2022 C++ Build Tools** with the
Windows SDK and **CUDA Toolkit 12.8**. Close the app and run:

```cmd
Setup_MAMMA.cmd
```

Review the upstream restricted-use terms before accepting the setup prompt.
Reopen the app, select **MAMMA**, and use **Model Manager** to download or import
its model files. MAMMA and locked-head SMPL-X assets require provider registration
and manual download. See the [MAMMA installation and usage guide](docs/MAMMA.md)
for prerequisites, exact files and storage requirements. This optional setup is
not needed for ExMR, FlowHMR or GVHMR.

### Install with an AI agent

After cloning, open this repository folder in a local coding agent such as Codex.
The [agent guide](docs/AGENT_GUIDE.md) covers setup, models, capture, retargeting,
projects and resources. For example, ask:

```text
Help me install and run RGB MoCap on this Windows PC.
Read AGENTS.md and docs/AGENT_GUIDE.md first. Check the GPU and free disk space,
run install.cmd, and launch RGBMoCap.cmd. Help me prepare the ExMR model files.
Preserve existing models, projects and settings, and keep the pinned runtimes.
Tell me when I need to sign in, accept provider terms or download a file manually.
```

For multi-camera capture, add:

```text
Also enable MAMMA. Follow docs/MAMMA_AGENT_SETUP.md to check the C++/CUDA tools,
run Setup_MAMMA.cmd after the required license review, and verify the native CUDA
modules. Help me prepare its models without changing the other model runtimes.
```

MAMMA's [agent setup guide](docs/MAMMA_AGENT_SETUP.md) includes build checks and
recovery. Provider login and license acceptance remain your responsibility;
do not put passwords or account tokens in the agent chat.

## Features

- Reconstruct one or several people and preview the body mesh and animation bones.
- Reduce sliding with foot/palm contact correction and editable contact timings.
- Retarget motion to Mixamo FBX, VRM and PMX humanoid characters.
- Export source animation as FBX; export characters as FBX with or without meshes,
  or PMX character motion as VMD for MMD.
- Save projects with separate results for each AI model.

## How it works

1. Select an AI model and prepare its files in **Model Manager**, then use
   **File > Open Video**. For MAMMA, select it before opening synchronized camera
   videos; thumbnails switch the preview and one timeline trim applies to all views.
2. For a single-video model, optionally use **Find person** to select people.
   MAMMA detects people during reconstruction; keep **VGGT (Automatic)** and
   enter an approximate **Reference Height (m)**, or import camera calibration.
   Click **Reconstruct 3D**.
3. Review the motion, edit contacts if needed and **Apply Correction**.
4. Export FBX, or import a character, retarget the motion and export it.

## Supported AI models

| Model / original project | Body representation | Fingers | Setup and use |
| --- | --- | --- | --- |
| [ExMR](https://github.com/twehrbein/ExMR) | SMPL-X | Included | [ExMR guide](docs/models/EXMR.md) |
| [FlowHMR](https://github.com/flowhmr/flowhmr) | SMPL-H | Included | [FlowHMR guide](docs/models/FLOWHMR.md) |
| [GVHMR](https://github.com/zju3dv/GVHMR) | SMPL-X | Optional HandFlow add-on | [GVHMR guide](docs/models/GVHMR.md) |
| [MAMMA](https://mamma.is.tue.mpg.de/) · multi-view motion capture | SMPL-X | Included | [MAMMA guide](docs/MAMMA.md) |

Requires 64-bit Windows, a compatible NVIDIA GPU/driver and OpenGL 4.3.
Model files are acquired separately; some require registration and license acceptance.

**Disk space:** allow about **45 GB (42 GiB)** for the app, runtimes,
Blender and model files for the three single-video models, including their optional
add-ons. The optional MAMMA runtime and model files are additional. Videos,
projects, imported characters, exports, generated caches and retained download
archives need additional space. See the [storage details](docs/USER_GUIDE.md#install).

[Cache and project cleanup: paths and deletion effects](docs/USER_GUIDE.md#storage-and-cleanup).

## Documentation

- [User guide: installation, controls and settings](docs/USER_GUIDE.md)
- [Model guides: files, defaults, options and limitations](docs/models/README.md)
- [MAMMA: additional installation, model files and multi-video workflow](docs/MAMMA.md)
- [Development and builds](docs/BUILD.md)
- [Agent operating and development guide](docs/AGENT_GUIDE.md)
- [Source validation](docs/VALIDATION.md)
- [Third-party notices and model terms](THIRD_PARTY_NOTICES.md)

Original RGB MoCap code and documentation use the [MIT License](LICENSE).
Third-party components and model files retain their own licenses.
