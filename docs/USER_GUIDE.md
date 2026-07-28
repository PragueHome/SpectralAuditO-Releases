# SpectralAuditO User Guide

## What is SpectralAuditO?

SpectralAuditO tells you the **true encoded quality** of your audio and video files. It looks inside each file's audio data rather than trusting the filename or container metadata, so it can detect fakes — files that claim to be 320 kbps MP3 or FLAC but were actually upscaled from a lower-quality source.

This guide is also available in PDF format: [SpectralAuditO-User-Guide.pdf](SpectralAuditO-User-Guide.pdf)

---

## Table of Contents

1. [Installation](#installation)
2. [First Scan](#first-scan)
3. [Understanding Results](#understanding-results)
4. [Waveform and Spectrogram View](#waveform-and-spectrogram-view)
5. [Searching and Filtering](#searching-and-filtering)
6. [File Operations](#file-operations)
7. [Exporting Reports](#exporting-reports)
8. [DJ Library Integration](#dj-library-integration)
9. [Settings](#settings)
10. [Command-Line Reference](#command-line-reference)
11. [Licensing](#licensing)
12. [Troubleshooting](#troubleshooting)

---

## Installation

### Windows (Standalone Build)

Download `SpectralAuditO-Setup.exe` from the [Releases page](https://github.com/PragueHome/SpectralAuditO-Releases/releases/latest) and run it. Standard Windows installer — no admin rights needed. Alternatively, download `specaudit-windows.zip` for a portable build: unzip and run `specaudit.exe`.

### macOS

Download `specaudit-macos.zip`, unzip it, and drag `SpectralAuditO.app` to your Applications folder. On first launch, you may need to right-click the app and select "Open" if you see a security warning (the app is unsigned).

### From Source (Developers)

```bash
git clone https://github.com/PragueHome/SpectralAuditO
cd SpectralAuditO
python -m venv .venv
.venv/Scripts/pip install -e .
.venv/Scripts/python -m specaudit
```

---

## First Scan

### GUI Walkthrough

1. **Launch the app** — you'll see an empty results table.
2. **Add folders** — click the folder icon or drag folders onto the window. A checkbox below lets you toggle recursive scanning (include subfolders).
3. **Start scanning** — click the Play button. The progress bar shows live progress; results appear row-by-row as they're analyzed.
4. **Review results** — each row is a file, color-coded by verdict:
   - **Green (OK)** — file is what it claims to be
   - **Orange (Suspicious)** — possible minor issue worth reviewing
   - **Red (Fake/Corrupt/Clipping)** — definite problem

### Command Line (Headless Batch)

```bash
python -m specaudit.cli path/to/music --csv report.csv --json report.json --html report.html
```

This scans a folder and exports results to CSV, JSON, and an interactive HTML report.

---

## Understanding Results

### Key Columns

**File** — the filename (full path shown in the tooltip)

**Quality** — verdict: OK, Suspicious, Fake, Corrupt, or Clipping

**Sample Rate** — the audio's sample rate (e.g., 44.1 kHz, 48 kHz) — hints at the source (CD = 44.1 kHz, streaming = 48 kHz or higher)

**Bitrate Tier** — estimated real bitrate tier (128k, 192k, 256k, 320k, lossless) — *actual* quality vs. the file's claimed bitrate

**Club-Grade** — 0-100% score (Wi-Fi-bar icon) rating how safe a track is for live playback, blending bitrate tier, true-peak headroom, suspicion flags, and loudness range

**Clipping** — percentage of audio peaks clipping (over 0dBFS), dangerous at volume

**Confidence** — how certain the verdict is (0-100%) — expressed as a signal-bar icon for visual consistency

**Detected Cutoff** (diagnostic column, hidden by default) — frequency where lossy compression was detected

**Flags** (diagnostic column, hidden by default) — detailed suspicion reasons

**BPM** (Elite feature) — detected tempo and beat-grid consistency (suffix `~` = variable tempo)

**Key** (Elite feature) — musical key in Camelot notation (open-key / MixedInKey style)

**Energy** (Elite feature) — 1-10 perceptual energy rating for DJ mixing workflows

### Fake Reasons

Files are flagged as **Fake** when:

- **Transcoded-up MP3** — low-bitrate source (e.g., 96k) re-encoded to higher (e.g., 320k); the spectrum reveals the original encoding
- **Lossy-in-FLAC** — MP3/AAC/Opus data wrapped in a FLAC container; FLAC is supposed to be lossless
- **Lossy-in-WAV** — same idea with WAV containers
- **Bitrate bump without quality** — file claims 320k but spectrum shows 128k
- **Soft-shelf suspicion** — a softer, wider lowpass shelf suggests resampling or codec-mismatch

---

## Waveform and Spectrogram View

### Waveform

Click a row to display the waveform — a real-time PCM visualization of the first audio segment. Green = left channel, red = right channel (stereo) or gray (mono).

### Spectrogram

Below the waveform is a time/frequency/intensity heatmap (Spek-style). Brighter = more energy. A sharp frequency cutoff (vertical line at high frequency) is the signature of lossy compression. A blue diagonal line shows the detected cutoff frequency.

### Interpreting the Spectrogram

- **Smooth, full spectrum** (all the way to Nyquist) — likely lossless or high-bitrate lossy
- **Sharp wall at 16 kHz or lower** — strong indicator of low-bitrate lossy (MP3, AAC, Opus)
- **Soft slope** — can be lossless or moderate-bitrate lossy; the verdict depends on the exact slope angle and persistence

---

## Searching and Filtering

Use the **Search** box to filter by filename (partial match). Use the **Status** dropdown to show only:

- **All** — every result
- **OK** — files that passed quality checks
- **Suspicious** — files with minor red flags
- **Flagged** — files with definite issues (Fake, Corrupt, Clipping)

Combine search + status filter to find, e.g., all suspicious FLAC files: search `*.flac`, status = Suspicious.

---

## File Operations

The **Actions** panel (bottom right) lets you:

### Rename

Rename flagged files to include the verdict, e.g., `Track.mp3` → `Track [FAKE 96k].mp3`. Includes a dry-run preview and a full undo log.

### Move

Move files to subfolders by verdict. The folder structure is created automatically.

### Copy

Bulk-copy files to an external drive or archive. Useful for exporting a cleaned library.

### Delete

Bulk-delete flagged files (with dry-run preview and undo).

### Fix Bitrate Tag

Overwrite the file's embedded bitrate metadata with the detected real bitrate. Works with MP3, FLAC, OggVorbis, and Opus; skips MP4-family containers (which don't persist custom metadata).

All operations show a **dry-run preview** first — review exactly what will happen before confirming. Every action is logged in an **Undo Journal** so you can roll back mistakes.

---

## Exporting Reports

**CSV** — tabular data, spreadsheet-friendly, includes every column from the results table

**JSON** — machine-readable, includes all metadata (tag-writing history, detected spectrum characteristics, DJ library sync status)

**HTML** — a self-contained interactive report with:
- Results table with sorting/filtering
- Analytics dashboard (bitrate distribution, verdict breakdown, clipping histogram)
- BPM/Key/Energy statistics (if musical metrics were enabled)
- Embedded spectrogram thumbnails for each flagged file (no external images)

---

## DJ Library Integration

SpectralAuditO can read your DJ library (Rekordbox, VirtualDJ, Serato, or M3U playlists) and write analysis results back into the library so flagged tracks are visible inside your DJ software.

### Loading a Library

In the **Settings** dialog, select **DJ Library** and choose your platform:

- **Rekordbox** — point to your Rekordbox database folder (usually `C:\Users\<Username>\AppData\Roaming\Pioneer\Rekordbox6\` on Windows)
- **VirtualDJ** — point to your VirtualDJ music folder or configuration
- **Serato** — point to your Serato crates folder
- **M3U Playlist** — point to a standard `.m3u` or `.m3u8` file

### Writing Results Back

After scanning, click **Sync to DJ Library** to write the detected bitrate tier and verdict into your library's metadata. Tracks will show the verdict in your DJ software's tag columns, making it easy to avoid flagged tracks on the fly.

Note: This feature requires a **Pro + DJ** licence (separate add-on).

---

## Settings

### Appearance

**Theme** — Light or Dark mode. Dark mode is easier on the eyes during late-night DJ sets.

**Window State** — the app remembers your window size and position between launches.

### Presets

**Analysis Presets** — quick-switch between predefined analysis depths:
- **Fast** — fewer segments sampled, faster scan, less accurate
- **Balanced** (default) — good speed/accuracy trade-off
- **Deep** — many segments, slower, higher confidence in edge cases

### Analysis Parameters

**Segment Duration** — how long each PCM sample is (in seconds). Longer segments are more robust to one-off glitches but may miss very short clipping events.

**Segment Count** — how many random segments to sample from the file. More segments = more accurate but slower.

**Corruption Check** — decode the entire file to check for decoding errors (slow but catches truncated files).

### Clipping Settings

**True Peak Ceiling** — the threshold above which audio is considered clipping. Default = 0 dBFS (full scale). Most mastering standards allow -1 to -3 dBFS headroom for streaming.

**Loudness Range** — LUFS (Loudness Units relative to Full Scale) range measurement. A high LRA means dynamic range; a very low LRA (< 4 LU) suggests over-compression.

### Musical Metrics (Elite)

**Enable BPM/Key/Energy Detection** — when checked, the scan includes tempo, musical key, and energy rating for every file (requires an Elite licence add-on). This adds a bit of time per file but unlocks the BPM/Key/Energy columns in results and reports.

### Logging

**Log Level** — capture diagnostic logs for troubleshooting. Default = Info. Set to Debug for very detailed output.

**Log Folder** — where log files are saved (used by support for troubleshooting).

### Media Cache

SpectralAuditO caches decoded PCM and spectrogram images to speed up repeated scans of the same files. You can clear the cache here if you're running low on disk space.

---

## Command-Line Reference

### Scan a folder and export reports

```bash
python -m specaudit.cli /path/to/music --csv report.csv --json report.json --html report.html
```

### Scan recursively and filter by verdict

```bash
python -m specaudit.cli /path/to/music -r --status fake,corrupt
```

### Write verdicts back to file tags

```bash
python -m specaudit.cli /path/to/music --write-tags
```

### Sync to DJ library after scanning

```bash
python -m specaudit.cli /path/to/music --sync-library rekordbox --library-path /path/to/rekordbox/db
```

### Full help

```bash
python -m specaudit.cli --help
```

---

## Licensing

SpectralAuditO uses a local-first licence model. Your licence is verified offline against a public key bundled with the app — no phone-home or internet required to use it.

### Free Tier

- Bulk scanning with quality detection
- CSV/JSON/HTML report export
- File operations (rename, move, copy, delete, tag fixes)
- 300-file-per-day scan cap

### Pro Licence

- Everything in Free, unlimited scans
- DJ library integration (Rekordbox, VirtualDJ, Serato, M3U)
- Musical metrics (BPM, key, energy) — shows as a separate **Elite** toggle in Settings

Available as:
- **One-time purchase** — perpetual licence, no expiry
- **Subscription** — annual renewal, includes free updates

### Pro + DJ Add-on

- Seamless integration with your DJ software
- Write verdicts back into Rekordbox/VirtualDJ/Serato libraries
- Deck-level monitoring (optional)

### Pro + Elite Add-on

- Full musical analysis suite (BPM with beat-grid, key detection, energy rating)
- Variable-tempo detection (rubato, tempo ramps)
- Advanced filtering and export by tempo/key

### Activating Online

1. In the app, go to **Help > Licence Info** or click the **Upgrade** prompt
2. Click **Activate** and sign in (or create an account)
3. Select your tier and add-ons
4. The app verifies your licence and activates it (internet required once)
5. After activation, the licence works offline

### Activating From File

If you don't have internet access on the machine where you're running SpectralAuditO:

1. On a machine with internet, go to **Help > Licence Info**
2. Click **Request Offline Licence**
3. Save the licence file (`*.sig`) to a USB drive
4. On the offline machine, go to **Help > Licence Info** and click **Activate from File**
5. Open the `.sig` file from the USB drive
6. The licence is now active offline (no expiry, unlimited use within your tier)

---

## Troubleshooting

### The app won't start

- Make sure you have Windows 10/11 (64-bit) or macOS 12+
- Download the latest version from the [Releases page](https://github.com/PragueHome/SpectralAuditO-Releases/latest)
- Check logs in `~/.specaudit/logs/` for error details

### Scan is very slow

- Use the **Fast** preset in Settings
- Reduce **Segment Count** to scan fewer parts of each file
- Disable **Corruption Check** (only needed for testing for truncation)
- On a network drive? Copy files locally first; network I/O is the bottleneck

### Results look wrong (e.g., a 320k MP3 marked as lossless)

- Make sure your FFmpeg is up to date (the app includes one by default)
- Set **Log Level** to Debug in Settings, re-scan, and check the logs in `~/.specaudit/logs/`
- File an issue on [GitHub](https://github.com/PragueHome/SpectralAuditO/issues) with the log attached

### Clipping percentages seem too high

- Clipping is measured as any peak over 0 dBFS in the decoded PCM. If your source file uses normalized floating-point (values > 1.0 in the PCM domain), that's expected.
- Adjust the **True Peak Ceiling** in Settings if your mastering standard uses headroom (e.g., -1 dBFS for streaming).

### DJ library sync isn't working

- Make sure your DJ software isn't actively using the library database — close Rekordbox/VirtualDJ/Serato first
- Check the log level and logs for error details
- Requires a **Pro + DJ** licence (free tier can't write back)

### Licence activation fails

- Check your internet connection
- Make sure your licence tier supports the features you're using
- If the activation server is down, try again later or use **Activate from File** (offline mode)

---

## Support

For bugs, feature requests, or support:

- **GitHub Issues**: [PragueHome/SpectralAuditO-Releases](https://github.com/PragueHome/SpectralAuditO-Releases/issues)
- **Email**: [salsedine@gmail.com](mailto:salsedine@gmail.com)

---

**SpectralAuditO** — true encoded quality, at a glance.
