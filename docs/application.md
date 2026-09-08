# OBS ROI Editor

OBS ROI Editor is an OBS Studio plugin for assigning encoder priority to parts
of the canvas. Positive priority favors an area; negative priority deprioritizes
it. It also provides a local preview of encoded output.

## Installation and package layout

Install the release package for the matching operating system while OBS is
closed, then start OBS and open **Tools**. The package contains the plugin
binary and its locale data; do not copy only the binary.

[OBS Studio 32.2.2](https://github.com/obsproject/obs-studio/releases/tag/32.2.2)
is the current Windows build target. Use the plugin package with that OBS
release or a compatible later release. The original 1.1.1
Windows binary imports older FFmpeg DLL names and does not load in OBS 32.2.x.

The build installs the plugin in the normal OBS plugin locations:

- Windows: `obs-plugins/64bit` and `data/obs-plugins/obs-roi-ui` below the OBS
  installation directory. The Windows installer uses the detected OBS install
  directory.
- Linux: `${CMAKE_INSTALL_LIBDIR}/obs-plugins` and
  `${CMAKE_INSTALL_DATAROOTDIR}/obs/obs-plugins/obs-roi-ui`.
- macOS: `~/Library/Application Support/obs-studio/plugins`, as an
  `obs-roi-ui.plugin` bundle containing its resources.

## Tools menu

The plugin adds two commands to **Tools**:

- **Region of Interest Editor** configures ROI regions for each scene.
- **Encoder Output Preview** encodes the current OBS video with a selected
  encoder and decodes it into a preview window. It can also expose the preview
  as a source in the current scene.

## Region of Interest Editor

Select a scene, add one or more regions, and select a region to edit it. Use
**Move Up** and **Move Down** to change the region order. The list marks a
disabled region with `DISABLED`.

Every region has:

- **Priority** from -100% to 100%. Zero is neutral.
- **Enabled**.
- **Smoothing**: None, Inside, Outside, or Edge. Smoothing adds stepped
  transition regions. Set its step count and outer/smoothing priority.

Region types are:

- **Scene Item Region** follows the selected scene item and therefore follows
  its position, size, transform, and visibility. Select the item from the
  scene-item list.
- **Manual Region** uses fixed pixel **Width**, **Height**, **X**, and **Y**.
- **Center Focus Region** builds a prioritized center area. Set the inner and
  outer radius, inner and outer step counts, outer priority, optional aspect
  correction, and optional circular inner shape. Set the center coordinates to
  `-1` to use the canvas center.

The editor includes an **ROI Encoder Map Preview**. Choose the encoder block
size and preview opacity. The map is colored by priority: green indicates a
positive value, red a negative value, and blue a value near zero.

**Enable Region of Interest feature** applies the configured regions. **Exclude
Recording Encoder** leaves the recording encoder out; streaming, replay buffer,
and the plugin preview output remain eligible. Changes are applied when the
scene or relevant output state changes.

## Encoder support and limits

The documented ROI encoder families are NVIDIA NVENC, AMD AMF, Intel QSV, and
x264. At runtime, the editor applies ROI only to encoders that advertise the
OBS ROI capability. A listed encoder or codec is therefore not a guarantee
that a particular installed encoder build supports ROI. Encoders may also
overshoot the target bitrate when ROI is used.

The block-size choices are 16, 32, 64, and 128 pixels. The labels identify
common combinations: 16 for H.264, 32 for NVENC/QSV HEVC, 64 for NVENC/QSV/AMF
AV1 and AMF HEVC, and 128 for AV1. Choose the size that matches the encoder
and codec in use.

Some encoders, including QSV, may not accept more than 256 regions. Smoothing
and center-focus steps can create many regions, so keep the total reasonable.
ROI coordinates are based on the base canvas. The plugin does not support
ROI-editor output scaling in **Settings > Video**; use the rescaling options
in **Settings > Output > Advanced** instead.

## Encoder Output Preview

1. Open **Tools > Encoder Output Preview**.
2. Select an available video encoder. Use **Refresh** after adding or changing
   an encoder.
3. Select **Start Preview** and wait for the first keyframe. The preview then
   shows the decoded encoded video and the input bitrate.
4. Select **Stop Preview** before changing the encoder.
5. To make the result available to multiview or a projector, select **Add
   Preview Source to current Scene**. The source is added to the current scene.

The preview can use recording or streaming encoders even when those outputs are
inactive. It supports H.264, HEVC, and AV1 when a matching FFmpeg decoder is
available. **Keep running when dialog is closed** leaves the preview running
when the dialog closes. Hardware decoding is present in the UI definition but
is currently hidden, so it is not a user setting in this version.

## Saved settings

OBS saves ROI configuration with the scene collection. ROI regions are stored
per scene, including their type, geometry, enabled state, priority, smoothing,
and center-focus values. The global ROI enabled state, recording exclusion,
block-map opacity, and window geometry are also saved.

The preview saves its background-running preference and window geometry. The
selected encoder is not saved; select it again after reopening the preview.

## Known limitations

- ROI has no effect on encoders that do not advertise OBS ROI capability.
- The encoder, driver, codec, and rate-control mode still determine the actual
  bitrate result; the priority value is guidance to the encoder, not a strict
  bitrate allocation.
- A hidden or missing scene item produces no active scene-item ROI.
- Excessive smoothing or center-focus steps can exceed an encoder's region
  limit.
- ROI editor scaling through **Settings > Video** is unsupported.
- The preview depends on a decoder for H.264, HEVC, or AV1 and may fail to
  start if the selected encoder or output cannot initialize.

## Source build overview

The project uses CMake and requires the OBS libraries (`libobs` and
`obs-frontend-api`), Qt Core and Widgets, and FFmpeg `avcodec`, `avutil`, and
`avformat`. CMake enables the OBS front-end and Qt integration, builds one
module from the C and C++ sources, and installs the `data/` locale resources.

The checked-in plugin version is 1.1.2.3322. `buildspec.json` pins OBS sources
at 32.2.2 and the Windows dependency packages dated 2026-08-26. The checked-in
preset is **windows-x64** (Visual Studio 17 2022, Qt 6); it uses an out-of-source
`build_x64` directory. In a prepared dependency environment, the normal flow
is:

```text
cmake --preset windows-x64
cmake --build --preset windows-x64
```

Other operating systems have platform-specific CMake install helpers, but no
matching configure/build preset is included in this source snapshot.
