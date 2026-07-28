# SpectralAuditO

Detects the *true* encoded quality of your audio and video files in bulk --
catches low-bitrate sources re-encoded to a higher bitrate, or lossy audio
wrapped in a lossless container, even when the file's own tags say otherwise.
Also flags corrupt files and clipping.

This repo carries **packaged, ready-to-run builds** and end-user documentation
only.

## Download

Grab the latest build from the [Releases page](https://github.com/PragueHome/SpectralAuditO-Releases/releases/latest):

- **Windows** -- `SpectralAuditO-Setup.exe` (installer, recommended) or
  `specaudit-windows.zip` (portable, no install needed)
- **macOS** -- `specaudit-macos.zip` (unzip and drag `SpectralAuditO.app` to
  Applications; first launch requires right-click -> Open if unsigned)
- **Linux** -- `specaudit-linux-<arch>.tar.gz` (unverified prebuilt tarball --
  build from source if you hit issues)

Current version: **1.1.0** (released 2026-07-28)

## Showcase

See SpectralAuditO in action: [SHOWCASE.md](SHOWCASE.md) or the
[PDF version](docs/SpectralAuditO-Showcase.pdf).

## User Guide

Full usage documentation: [USER_GUIDE.md](docs/USER_GUIDE.md) or the
[PDF version](docs/SpectralAuditO-User-Guide.pdf).

## System Requirements

- Windows 10/11 (64-bit) or macOS 12+
- No separate ffmpeg install needed -- it's bundled
- No admin rights required for the Windows portable zip; the installer needs
  standard install permissions

## License / Support

See the in-app Help menu for licensing and activation details, or the
[User Guide](docs/USER_GUIDE.md)'s Licensing section. For support or bug
reports, open an issue on this repo.
