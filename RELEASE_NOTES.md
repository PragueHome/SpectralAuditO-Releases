## SpectralAuditO {{VERSION}}

Released {{DATE}} ({{TAG}}).

### What's new

- Windows distribution is now signed end to end, including the MSIX package, improving trust and reducing SmartScreen/Defender warnings for installer-based installs.
- New Club-Grade scoring makes it easier to judge whether a file is safe to play at volume at a glance.
- Optional BPM / key / energy metrics remain available as separate columns and exports for more detailed workflow analysis.
- Video bitrate reporting is more accurate for containers whose audio stream lacks its own bitrate metadata.

### Downloads in this release

{{ARTIFACTS_LIST}}

### Package options

Choose the artifact that fits your workflow:

- `specaudit-windows.zip` — portable Windows build for users who prefer to unpack and run the app manually.
- `SpectralAuditO-Setup.exe` — installer-based Windows build for standard installs.
- `SpectralAuditO.msix` — Microsoft Store-style Windows package, signed and ready for modern deployment workflows.
- `specaudit-macos.zip` — macOS app bundle packaged as a ZIP archive for manual installation and distribution.

### Documentation

- User Guide: {{GUIDE_LINK}}
- Showcase: {{SHOWCASE_LINK}}

### Verification

⚠️ **These builds are not code-signed or notarized.** The OS warnings you may see are expected for unsigned PyInstaller builds.

SHA-256 checksums (also attached as `CHECKSUMS.txt`):

```text
{{CHECKSUMS}}
```

{{VIRUSTOTAL_STATUS}}
