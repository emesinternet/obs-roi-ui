# OBS 32.2.2 update report

## Result

The Windows plugin now builds against OBS Studio 32.2.2 and loads in a clean
OBS Studio 32.2.2 portable runtime. The compatibility fix is released as
version 1.1.2.3322 and changes no user-facing feature or data format.

## Cause

The released 1.1.1 Windows DLL imports `avcodec-61.dll` and `avutil-59.dll`.
OBS Studio 32.2.2 ships `avcodec-62.dll` and `avutil-60.dll` because its
bundled FFmpeg version changed. Windows therefore rejects the old plugin DLL
before OBS can call its module entry point. OBS logged loader error 126 and did
not load the module.

The supplied OBS runtime archive is OBS Studio 32.2.2. Its SHA-256 is:

```text
4D6E40E3AB155F56B30DE517380566A206D74B63CDF5AD49AA596924768F97E1
```

[PR #26](https://github.com/derrod/obs-roi-ui/pull/26) identified the same
FFmpeg import problem. It also identified a Qt display callback lifetime
problem. Both findings were checked against this source tree before changing
the code.

## Review of PR #26

PR #26 is correct about the FFmpeg break and the required OBS 32 build-system
updates. It also contains a useful Qt lifetime fix, which is included here.
The PR refreshes the full platform template and changes warning-label theme
properties. Those broad template changes and non-load-related visual changes
were not needed for this Windows fix, so they were not copied.

## Changes

- Updated `buildspec.json` to OBS sources 32.2.2 and the 2026-08-26 Windows
  dependency and Qt 6 packages, with verified SHA-256 values.
- Updated the CMake minimum version and Qt targets for the OBS 32.2.2 build
  system: `Qt6::Core` and `Qt6::Widgets`.
- Changed the OBS configure flags from the removed `ENABLE_UI` option to
  `ENABLE_FRONTEND` and set the OBS CMake API version to 3.0.0.
- Updated the Windows preset to use the installed Windows 10 SDK 10.0.26100
  and read the Visual Studio platform name correctly when the SDK version is
  part of the preset architecture value.
- Changed Qt checkbox signal connections from the deprecated
  `stateChanged` signal to `checkStateChanged`, required by Qt 6.11 with
  warnings treated as errors.
- Added the widget as the receiver for the preview display's Qt signal
  connections. Qt now disconnects those callbacks when the widget is deleted.
  This prevents callbacks into a destroyed preview widget.
- Removed two unused CMake version assignments made obsolete by the new OBS
  configure command.

No new architecture, compatibility layer, fallback FFmpeg loading, or
performance change was added.

## Verification

The following checks passed on Windows:

1. `cmake --build --preset windows-x64 --parallel` and the warnings-as-errors
   `windows-ci-x64` preset completed successfully with Visual Studio 17 2022,
   MSVC 14.44, and Windows SDK 10.0.26100.
2. The rebuilt DLL is x86-64 and imports `avcodec-62.dll` and `avutil-60.dll`.
   It also imports `obs.dll`, `obs-frontend-api.dll`, and the Qt 6 runtime.
   The final DLL SHA-256 is
   `E9BE6792B93145859E22AADF6C9D18DE94F2D360E010AC910C8AAEADB3BEF714`.
3. The original 1.1.1 DLL imports `avcodec-61.dll` and `avutil-59.dll`.
4. In a clean portable OBS 32.2.2 runtime, the original DLL produced:

   ```text
   LoadLibrary failed for '../../obs-plugins/64bit/obs-roi-ui.dll': The specified module could not be found.
   Module '../../obs-plugins/64bit/obs-roi-ui.dll' not loaded
   ```

5. In a clean portable OBS 32.2.2 runtime, the rebuilt and installed package
   produced:

   ```text
   Loading module: obs-roi-ui.dll
   output 'encoder_preview' (encoder_preview) created
   private source 'Encoder Output Preview' (encoder_preview_source) created
   [obs-roi-ui] plugin loaded successfully (version 1.1.2.3322)
   ```

The runtime test used separate clean OBS directories for the old and new
packages. Both OBS processes remained responsive. The old process stayed
running with the plugin skipped; the new process stayed running with the
plugin loaded and its output and source registered.

The exact log files are `validation/obs-old-runtime/config/obs-studio/logs/2026-09-07 19-57-38.txt`
and `validation/obs-new-runtime/config/obs-studio/logs/2026-09-07 20-01-16.txt`.

## Scope

This update fixes the OBS 32.2.x load failure and the Qt callback lifetime
issue. It does not change ROI calculations, encoder selection, preview
encoding, saved settings, or performance. No profiling was needed because no
performance code changed.

The built Windows package is staged by CMake at `validation/install-new` during
verification. It contains `obs-roi-ui.dll`, its PDB, and the locale resource
under the normal OBS plugin paths.
