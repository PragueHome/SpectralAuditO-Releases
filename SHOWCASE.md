# SpectralAuditO Showcase

![SpectralAuditO Logo](docs/screenshots/icon.png)

## See through fake audio quality

See through fake audio quality. Bulk-scan your music and video library with real spectral analysis -- catch upscaled MP3s, lossy-in-lossless fakes, clipping, and corrupt files before they ruin a set.

---

## Open-source foundation, proprietary detection

SpectralAuditO is built on proven open-source tools -- **FFmpeg** for audio decoding, **NumPy** for the FFT, and **Qt** (via PySide6) for the desktop interface -- and makes no claim to have invented them.

The layer that sits on top is what makes the results reliable:

- **Slope-based cutoff detection** -- scans from Nyquist downward for a brick-wall frequency shelf, not a fixed dB floor, so it correctly distinguishes a gentle high-frequency rolloff in natural audio from the hard shelf a lossy encoder leaves behind. This is what prevents false positives on genuine lossless files -- the costliest possible error in audio quality auditing.
- **Soft-shelf second pass** -- a wider, more forgiving scan catches the subtler shelves that low-bitrate AAC and similar codecs leave when their hard knee is gentle enough to slip past the strict pass.
- **Per-codec tier tables and segment-sampled decoding** -- codec-specific bitrate-tier lookups combined with a bounded-segment decode strategy keep large-batch scans both fast and format-accurate.
- **Club-Grade scoring model** -- a configurable weighted composite of tier position, true-peak headroom, suspicion flags, and loudness range, calibrated so users get one actionable number rather than four independent signals to weigh manually.
- **Musical metrics pipeline** -- BPM/beatgrid, musical key (with Camelot-wheel notation), and a perceptual energy rating, all built on the same decoded audio used for quality analysis.

The open-source components handle the heavy lifting. The proprietary tuning and detection logic are what make the verdicts trustworthy.

---

## Real detection, not guesswork

SpectralAuditO decodes the actual audio and runs spectral (FFT) analysis to find the frequency cutoff a lossy encoder leaves behind -- the tell-tale sign of a file that's been upscaled or mislabeled, no matter what the container metadata claims.

### Key Features

**Fake & quality detection**
Finds the real encoded bitrate tier from the spectrum itself, flags transcoded-up files and lossy-in-lossless fakes, and catches truncated/corrupt files and clipping.

**Graduated Club-Grade score**
A 0-100% score -- shown as Wi-Fi-style signal bars -- rating how safe a track is to play at volume, blending bitrate tier, true-peak headroom, suspicion flags, and loudness range into one number.

**Musical metrics (Elite)**
Opt-in BPM/beatgrid, musical key with Camelot-wheel notation, and a 1-10 perceptual energy rating -- shown as separate columns and exported to every report.

**DJ library integration**
Scan straight from Rekordbox, VirtualDJ, or Serato, and write verdicts back into your library so flagged tracks are visible right in your DJ software.

**File operations, done safely**
Rename, move, copy, delete, or fix bitrate tags in bulk -- every action shows a dry-run preview first, and a full undo journal has your back.

**CSV / JSON / HTML reports**
Export every metric to CSV or JSON for your own analysis, or a self-contained HTML report with an analytics dashboard and embedded spectrogram thumbnails.

---

## In Action

### Main Results Window

Scan results with spectrogram viewer, quality detection, Club-Grade scoring, and clipping indicators.

![Main Window](docs/screenshots/main_window.png)

### Settings & Configuration

Configure analysis depth, musical metrics, DJ integration, and visual theme.

![Settings Dialog](docs/screenshots/settings_dialog.png)

---

## Choose Your Tier

### Free ($0)
- 50 files per session
- Full detection engine
- Spectrogram viewer

### Pro (One-time / subscription)
- Unlimited scanning
- CSV / JSON / HTML export
- Rename, move, copy, delete, tag writing

### Pro + DJ (Add-on)
- Everything in Pro
- Rekordbox / VirtualDJ / Serato read & write-back

### Pro + Elite (Add-on)
- Everything in Pro
- BPM / key / energy detection

---

## System Requirements

- Windows: 10/11 (64-bit)
- macOS: 12+ (Intel or Apple Silicon)
- No separate ffmpeg needed - it's bundled
- No admin rights required for portable versions

---

## Get Started

Visit the [Downloads](https://github.com/PragueHome/SpectralAuditO-Releases/releases/latest) page to grab the latest build for your platform.

For detailed usage instructions, see the [User Guide](docs/USER_GUIDE.md).

---

**SpectralAuditO -- true encoded quality, at a glance.**

For support or licensing inquiries, contact PragueHomeServer@gmail.com
