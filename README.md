# OBS ROI / Encoder Output Preview

This fork is based on the [original OBS ROI / Encoder Output Preview repository](https://github.com/derrod/obs-roi-ui). I intend to keep it up to date with current OBS releases.

<!-- obs-compatibility-badge:start -->
[![OBS Studio compatibility](https://img.shields.io/badge/OBS_Studio-32.2.2_compatible-brightgreen)](https://github.com/obsproject/obs-studio/releases/tag/32.2.2)
<!-- obs-compatibility-badge:end -->

[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](LICENSE)

#### A plugin for encoding nerds.

This plugin adds two features to OBS Studio. Both are available from the
**Tools** menu:

1. Region of Interest Editor
2. Encoder Output Preview

## Region of Interest Editor

![Region of Interest Editor](repo/screenshot.png)

The ROI editor lets you assign encoder priority to selected areas of the
canvas. Positive priority favors an area. Negative priority deprioritizes it.

### Features

- Per-scene configuration
- Scene item regions that follow an item
- Manual regions with custom size and position
- Center Focus regions with rectangular or radial shapes
- Smoothing, encoder map preview, and configurable priority

### Encoder support

- NVIDIA NVENC
- AMD AMF
- Intel QSV
- x264

The encoder must advertise OBS ROI support. The priority is guidance to the
encoder, not a strict bitrate allocation.

## Encoder Output Preview

![Encoder Output Preview](repo/screenshot_preview.png)

The Encoder Output Preview shows the result of an encoder without starting a
stream or recording. It is useful for checking ROI changes and encoder
settings.

### Features

- Preview recording and streaming encoders while outputs are inactive
- Add the preview as a source for scenes, multiview, and projectors
- H.264, AV1, and HEVC preview support when a decoder is available

## Download

The [Releases](https://github.com/emesinternet/obs-roi-ui/releases) page
contains the Windows x64 installer and ZIP package. GitHub also provides
source code ZIP and tar archives for every tagged release.

## Build

The project uses CMake. On Windows, use the **windows-x64** preset:

```text
cmake --preset windows-x64
cmake --build --preset windows-x64
```

## License

This project is licensed under the GNU General Public License, version 2.
See [LICENSE](LICENSE).
