# SpectralAuditO

Detects the *true* encoded quality of your audio and video files in bulk --
catches low-bitrate sources re-encoded to a higher bitrate, or lossy audio
wrapped in a lossless container, even when the file's own tags say otherwise.
Also flags corrupt files and clipping.

Built on proven open-source components (FFmpeg for decoding, NumPy for the FFT, Qt/PySide6 for the GUI). The proprietary layer is the detection logic: a slope-based spectral cutoff algorithm that avoids false positives on genuine lossless audio, a soft-shelf second pass for subtle low-bitrate codecs, per-codec tier tables, segment-sampled decoding, the Club-Grade composite scoring model, and the BPM/key/energy musical metrics pipeline.

This repo carries **packaged, ready-to-run builds** and end-user documentation
only.

## Download

Grab the latest build from the [Releases page](https://github.com/PragueHome/SpectralAuditO-Releases/releases/latest):

- **Windows** -- `SpectralAuditO-Setup.exe` (installer, recommended) or
  `specaudit-windows.zip` (portable, no install needed)
- **macOS** -- `specaudit-macos.zip` (unzip and drag `SpectralAuditO.app` to
  Applications; first launch requires right-click -> Open if unsigned)

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

## ⚠️ Security Notice: Unsigned Builds

**Current builds are NOT code-signed or notarized.** This means:

### Windows

- **SmartScreen warning** — Windows Defender SmartScreen may warn "Unknown Publisher"
  on first run. This is normal for unsigned software and **does not indicate a security
  issue** — it's a reputation system that new applications accumulate over time.
- **The warning can be dismissed** — click "More info" → "Run anyway" to proceed
- **Portable zip** — extracted `specaudit.exe` can be run directly without any warnings
  (SmartScreen only warns on downloaded executables)

### macOS

- **Gatekeeper on first launch** — macOS may show "cannot be opened because the
  developer cannot be verified." This is expected for unsigned software.
- **Workaround** — right-click the `SpectralAuditO.app` and select "Open" to bypass
  the warning (you only need to do this once)
- **Trust not affected** — Gatekeeper's warning does **not** indicate malware; it's
  simply that Apple couldn't verify the code signature

### Verification & Trust

To verify the integrity of downloaded artifacts:

1. **Checksums (SHA-256)** — every release includes a `CHECKSUMS.txt` asset listing
   the SHA-256 hash of each package. After downloading, verify the hash matches:
   - Windows (PowerShell): `Get-FileHash .\specaudit-windows.zip -Algorithm SHA256`
   - macOS: `shasum -a 256 specaudit-macos.zip`

   Compare the output against the matching line in `CHECKSUMS.txt` from the same
   release. A mismatch means the file was corrupted or tampered with — do not run it.

   Current v1.1.0 checksums (also attached as `CHECKSUMS.txt` on the
   [release page](https://github.com/PragueHome/SpectralAuditO-Releases/releases/tag/v1.1.0)):

   ```text
   a2ea4fd34ad0f35b6c703ab21041054fa00dc9851d3e1de06b05a7ca7c27066b  SpectralAuditO-Setup.exe
   ce9f44fe4196df36449360aec80617981a7cd26237e3094d7c4dd1e036a4bc79  specaudit-windows.zip
   1ad3a625dfd7ad37b375d72c6a7b047b13fd95d1e6efddda9909128a6756b1ac  specaudit-macos.zip
   ```
2. **VirusTotal scans** — each release's notes link to a
   [VirusTotal](https://www.virustotal.com/) report for every attached binary/installer,
   showing results from 70+ antivirus engines. You can also submit a downloaded file
   yourself at [virustotal.com/gui/home/upload](https://www.virustotal.com/gui/home/upload)
   and compare its SHA-256 to the one in `CHECKSUMS.txt` to confirm you scanned the exact
   official artifact.

   Current v1.1.0 scans:
   - Windows installer (`SpectralAuditO-Setup.exe`): [VirusTotal report](https://www.virustotal.com/gui/file/a2ea4fd34ad0f35b6c703ab21041054fa00dc9851d3e1de06b05a7ca7c27066b)
   - Windows portable zip (`specaudit-windows.zip`): [VirusTotal report](https://www.virustotal.com/gui/file/ce9f44fe4196df36449360aec80617981a7cd26237e3094d7c4dd1e036a4bc79)
   - macOS zip (`specaudit-macos.zip`): [VirusTotal report](https://www.virustotal.com/gui/file-analysis/NTZkMzhkMjBhMjFjYTQxMTI5ZGI2ODFkMjhiOTlkMGQ6MTc4NTIzNTU5NQ==)

   **A stray detection or two on these reports is expected and does not mean the file
   is malicious.** PyInstaller-built Python applications are a well-known source of
   antivirus false positives, for reasons that have nothing to do with the app's actual
   behavior:
   - The executable is a **self-extracting bundle** (PyInstaller unpacks a Python
     interpreter and all dependencies into a temp directory at launch) — the same
     runtime-unpacking pattern some heuristic/generic-detection engines use as a
     malware signal, regardless of what the unpacked code does.
   - It is **unsigned**, so engines that factor "no publisher signature + newly seen
     hash" into their heuristic score will rate it more cautiously than an identical,
     signed binary.
   - It has **very few prior submissions** to VirusTotal — the same low-reputation
     effect that drives the Windows SmartScreen warning above also nudges a handful of
     purely heuristic (not signature-based) engines.

   If you want a second opinion beyond the aggregate score, check *which* engines flag
   it on the report page — a hit from a well-known signature-based engine (naming a
   specific known malware family) would be worth investigating, whereas a handful of
   generic heuristic labels (e.g. "Gen:Heur", "PUA", "Malicious.Moderate") clustered
   around the "unsigned Python bundle" engines is the expected pattern.

### Planned: Code Signing (Future)

Code signing and notarization infrastructure is **prepared and ready** for deployment:

- Windows: `scripts/sign_windows.ps1` (Authenticode signing) — requires EV code-signing
  certificate
- macOS: `scripts/sign_macos.sh` (Apple Developer signing) + `scripts/notarize_macos.sh`
  (Apple notarization) — requires Apple Developer Program membership

Once signed builds are deployed, users will see:

- Windows: Developer name in SmartScreen (reputation system still applies, but no "Unknown Publisher" warning)
- macOS: Verified developer name in Gatekeeper (no warnings on first launch)

## License / Support

See the in-app Help menu for licensing and activation details, or the
[User Guide](docs/USER_GUIDE.md)'s Licensing section. For support or bug
reports, open an issue on this repo.
