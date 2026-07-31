# SpectralAuditO — User Guide

SpectralAuditO tells you the *true* encoded quality of your audio and video files. It looks
inside each file's audio data rather than trusting the filename or container metadata, so it
can detect fakes — files that claim to be 320 kbps MP3 or FLAC but were actually upscaled from
a lower-quality source.

### What’s new in 1.2.0
This release focuses on making the app easier to trust and easier to use:
- **Signed Windows installers and MSIX** — the Windows installer and MSIX package are now signed as part of the build pipeline, improving trust and reducing install-time warnings.
- **Club-Grade score** — a single 0–100% playability score, shown as Wi‑Fi bars in the results table.
- **Optional musical metrics** — BPM, musical key, and energy can be shown as separate columns when enabled.
- **Clearer support and licensing** — licence requests and support contact are surfaced more consistently.
- **Better video handling** — claimed bitrate for video containers is reported more accurately.

> This guide is also available inside the app: **Help → User Guide**. **Help → What's New /
> Showcase** shows a visual tour of the app's features.

---

## Table of Contents

1. [Installation](#1-installation)
2. [First scan](#2-first-scan)
3. [Understanding results](#3-understanding-results)
4. [The waveform and spectrogram view](#4-the-waveform-and-spectrogram-view)
5. [Searching and filtering](#5-searching-and-filtering)
6. [File operations](#6-file-operations)
7. [Exporting reports](#7-exporting-reports)
8. [DJ library integration](#8-dj-library-integration)
9. [Settings](#9-settings)
10. [Command-line reference](#10-command-line-reference)
11. [Licensing](#11-licensing)
12. [Troubleshooting](#12-troubleshooting)

---

## 1. Installation

### Windows (standalone build)

Two options are available — both contain identical software:

#### Option A — Installer (recommended)

1. Download `SpectralAuditO-Setup.exe` and run it.
2. No admin/UAC prompt — installs per-user under `%LocalAppData%\Programs\SpectralAuditO`.
3. Creates a Start Menu shortcut (and optionally a Desktop shortcut).

#### Option B — Portable zip

1. Download `specaudit-windows.zip` and unzip it anywhere you like.
2. Run `specaudit.exe` directly — no install step needed.

Both options are Authenticode-signed. Windows may still show a **SmartScreen prompt**
("Windows protected your PC") on first launch — click **More info → Run anyway** to
proceed. This prompt appears because SmartScreen builds reputation for new releases over
time; it is not an indication of malware. See [§12 Troubleshooting](#12-troubleshooting)
if you would prefer to verify the signature yourself, or if Defender quarantines the file.

### From source (developers)

```
python -m venv .venv
.venv\Scripts\pip install -e .[dev]
.venv\Scripts\python -m specaudit        # GUI
.venv\Scripts\python -m specaudit.cli    # CLI
```

FFmpeg must be available (either already in `specaudit/bin/`, on `PATH`, or pointed to by
`SPECAUDIT_FFMPEG_DIR`).

---

## 2. First scan

### GUI

1. Launch `specaudit.exe`.
2. **Add files or folders** — drag them onto the file list on the left, or click the folder
   icon in the toolbar. Subfolders are scanned recursively by default; uncheck **Recursive**
   next to the file list to scan only the top level.
3. Click **Scan** (▶ in the toolbar). Results appear as each file completes — you don't have
   to wait for the whole batch.
4. Click **Stop** (■) at any time to cancel; files already analyzed are kept.

### CLI (headless / batch)

```sh
specaudit path\to\music --csv report.csv --html report.html
```

Use `--help` for the full option list, or see [§10 Command-line reference](#10-command-line-reference).

---

## 3. Understanding results

Each row in the results table represents one file. The **Status** column gives the verdict:

| Status | Colour | Meaning |
|---|---|---|
| **OK** | white | Passes all checks. The detected frequency cutoff matches the claimed encoding. |
| **OK [flagged]** | pale amber | Technically passes, but something is mildly suspicious (e.g. stereo channels are near-identical, or texture is unusually flat). Not a definitive fake — treat as a prompt to double-check. |
| **FAKE** | red | The spectrum reveals a brick-wall cutoff well below what the claimed bitrate or codec would produce. The file was almost certainly upscaled from a lower-quality source. |
| **CORRUPT** | orange | The decodable audio duration differs significantly from the duration reported in the file header. The file may be truncated or internally damaged. |
| **CLIPPING** | yellow | One or more samples are at or above 0 dBFS, or the true peak (inter-sample) exceeds 0 dBTP. |
| **INCONCLUSIVE** | grey | The audio contains too little useful signal (silence, very short clip, extreme spectral flatness) to make a determination. |
| **ERROR** | grey | The file could not be decoded at all (unsupported format, corrupt header, ffmpeg failure). |

Hover over any cell in the **Status** column to see a tooltip with the specific reason.

### Other columns

| Column | Description |
|---|---|
| File | Full path to the file |
| Container / Codec | What the file *claims* to be (from its metadata) |
| Claimed bitrate | The bitrate or lossless flag the container reports |
| Detected cutoff | The highest frequency with useful energy, in kHz — the key diagnostic number |
| Estimated real | The bitrate tier SpectralAuditO estimates based on the detected cutoff. A lossy codec with no detected shelf shows **full-band**; only a genuinely lossless codec shows **lossless/full-band**. |
| Confidence | How certain the detection is (0–100 %) |
| Club-Grade | A graduated **0–100 % score**, shown as Wi-Fi-style signal bars, rating how safe the file is to play at volume — blends bitrate tier, true-peak headroom, suspicion flags, and loudness range/LRA into one number. Hover for a full breakdown plus a separate strict pass/fail check. `FAKE`/`CORRUPT`/`CLIPPING` files score **0%**; files with no usable signal (`INCONCLUSIVE`/`ERROR`) show **N/A** rather than a misleading 0. |
| BPM / Key / Energy | Detected tempo, musical key (with Camelot-wheel notation), and a 1–10 perceptual energy rating. Blank unless **Musical metrics (Elite)** is enabled — see §9 Settings. |
| Notes | Any extra detail: fake reason, clipping stats, true peak, suspicion flags |

### Fake reasons

When the status is **FAKE**, the Notes column explains why:

- **Transcode up** — the file is already a lossy format (e.g. MP3), but a lower-bitrate lossy
  shelf was found inside it, indicating it was re-encoded upward from a worse source.
- **Lossy in lossless** — the container is a lossless format (FLAC, WAV, ALAC), but the
  spectrum shows a lossy encoder's fingerprint inside.
- **Sample-rate upscale** — the sample rate appears to have been artificially increased (e.g.
  44.1 kHz content presented as 96 kHz).

---

## 4. The waveform and spectrogram view

Click any row to open the detail panel (right side of the window). It contains two stacked
visualisations separated by a draggable divider — the waveform on top and the spectrogram
below. Both update whenever you select a different row.

### Waveform

The waveform shows amplitude over time as three overlapping colour-coded frequency bands,
inspired by Mixxx's "Filtered" waveform style:

| Colour | Band | Frequency range |
| --- | --- | --- |
| Red | Bass | below 300 Hz |
| Green | Mids | 300 Hz – 4 kHz |
| Blue | Highs | above 4 kHz |

Each band is independently normalised so even tracks with very little high-frequency content
show clearly. A grey underlay shows the unfiltered amplitude envelope.

The waveform is generated by decoding the full audio track, splitting it into chunks, and
computing the peak magnitude in each frequency band per chunk — no scipy or external DSP
library required, just numpy FFTs.

### Spectrogram

The spectrogram shows a time/frequency heatmap of the selected file — the same view as Spek.
Brighter areas indicate more energy. A horizontal cyan line marks the detected cutoff
frequency.

A brick-wall lossy cutoff is immediately visible as a hard horizontal line above which the
display goes dark. A genuine full-band file will have energy (even faint energy) all the way
up to the top.

Both the waveform and spectrogram render in the background — selecting a different row
cancels any in-progress render so the view always reflects the current selection.

---

## 5. Searching and filtering

**Search box** (toolbar) — type to filter the results table by filename or status text. The
filter is live; clearing the box restores all rows.

**Status filter** (dropdown next to search) — limit the table to a specific status (All, OK,
Fake, Corrupt, Clipping, Inconclusive, Error).

Column headers are clickable to sort. Click again to reverse order.

---

## 6. File operations

File operations appear in the **Actions** panel (bottom of the window). All operations show a
**dry-run preview** listing every planned action before anything is changed. You must click
**Apply** in the preview dialog to proceed.

> File operations require a **Pro licence**. Buttons are greyed out on the Free tier.

| Button | What it does |
|---|---|
| **Rename selected** | Renames selected files in-place according to the naming pattern (configured in Settings). |
| **Move selected…** | Moves selected files to a folder you choose. |
| **Copy selected…** | Copies selected files to a folder you choose, leaving originals intact. |
| **Delete selected** | Sends selected files to the Recycle Bin (not a permanent delete). |
| **Fix bitrate tag…** | Writes SpectralAuditO's detected real bitrate into the file's embedded tags. |
| **Write verdict to tags…** | Stamps the scan verdict (OK / FAKE / …) into the Grouping or Comment tag of each selected file. |
| **Sync to DJ library…** | Writes the verdict stamp directly into a Rekordbox / VirtualDJ / Serato database (requires a DJ library to be loaded — see §8). |
| **Find duplicates…** | Identifies duplicate tracks across the scanned files, ranked by spectral quality. Report only — nothing is deleted. |
| **Undo last** | Reverses the last file operation batch (rename/move/copy/delete/tag write). |

---

## 7. Exporting reports

From the **File** menu or the toolbar, choose **Export → CSV / JSON / HTML**.

> Exports require a **Pro licence**.

| Format | Best for |
|---|---|
| **CSV** | Importing into Excel, scripting, or any data analysis tool. 39 columns covering every metric, including the Club-Grade score and (when enabled) BPM/key/energy. |
| **JSON** | Programmatic consumption; includes full confidence and Club-Grade score breakdowns per file. |
| **HTML** | Sharing a visual report. Self-contained single file with a live search filter, an analytics dashboard (status/codec/loudness/Club-Grade distributions), and embedded spectrogram thumbnails for flagged files. |

---

## 8. DJ library integration

> DJ library features require a **Pro licence with DJ entitlement**.

SpectralAuditO can scan directly from a DJ library database instead of (or in addition to) a
folder path, and can write results back into the library.

### Loading a library (GUI)

Use **File → Open library** and select the appropriate source:

| Library | What to select |
|---|---|
| **Rekordbox** | The XML export (Rekordbox → File → Export Collection) |
| **VirtualDJ** | `database.xml` (usually in `Documents\VirtualDJ`) |
| **Serato** | The `_Serato_` folder or its parent |
| **M3U / M3U8** | Any `.m3u` or `.m3u8` playlist file |

After loading, use the **Playlist** dropdown to restrict to a specific playlist or crate
(Rekordbox and Serato only). Then click **Scan**.

### Writing results back

After a scan, select rows and click **Sync to DJ library…** to write the verdict stamp into
the library's own database. A dry-run preview is always shown first.

---

## 9. Settings

Open **Edit → Settings** (or the gear icon). All settings are saved automatically when you
click **OK** and restored the next time the app launches — nothing is ever lost between
sessions. Settings are stored in `~/.specaudit/analysis_config.json`.

> **First launch:** if you open SpectralAuditO for the first time with an **Elite** licence,
> **BPM / key / energy detection** is automatically switched on so you don't need to hunt for
> the checkbox. Once you save any settings of your own, your choices stick permanently.

### Appearance

| Setting | Description |
|---|---|
| **Theme** | `System` (default) — follows the OS light/dark mode. `Light` or `Dark` force a fixed palette. |

### Presets

The **Presets** dropdown lets you instantly apply a named set of analysis parameters — useful
when you regularly switch between different scan scenarios. Four built-in presets are always
available:

| Preset | Best for |
|---|---|
| **DJ Library Bulk Scan** | Fast pass over a large library — 16 segments × 2 s, auto deep-checks, full-file clipping off. Balances speed and accuracy for routine quality-control. |
| **Mastering / Label QC** | Highest thoroughness — 24 segments × 3 s, deep checks always on, full-file clipping pass enabled. Catches even subtle upscales and every clip. |
| **Classical & Acoustic** | Dynamically complex material — wider shelf-persistence tolerance and a conservative silence gate so quiet passages aren't silently skipped. True peak ceiling at −3 dBTP. |
| **Electronic / Club** | High-energy club tracks — tighter true peak ceiling (−0.5 dBTP), per-channel clip analysis, and a −50 dBFS silence gate tuned for dense mixes. |

You can also click **Save as…** to store the current settings under your own name, then recall
them at any time. User-saved presets can be deleted; built-in presets cannot.

### Analysis parameters

These are the core knobs that control how thoroughly and how sensitively each file is analysed.
The defaults work well for most libraries; adjust them if you consistently see false positives
or false negatives on a particular kind of material.

| Setting | Default | Description |
|---|---|---|
| **Segments sampled per file** | 8 | Number of time windows decoded for spectral analysis. More segments catch fakes that are hidden in only part of a track; fewer is faster. |
| **Segment duration (s)** | 2.0 | How many seconds each segment covers. Longer segments give a more stable FFT average; shorter segments make large-batch scans faster. |
| **Shelf drop threshold (dB)** | 25 | How sharply the spectrum must fall within the detection band to count as a brick-wall shelf. **Lower = more sensitive** (catches softer cutoffs, may increase false positives on material with natural roll-off). **Higher = stricter** (only hard, obvious shelves trigger FAKE). |
| **Shelf persistence (Hz)** | 300 | How wide the region above the detected knee must stay low before the shelf is confirmed. Increase to require a wider flat dead zone; decrease to catch narrow-band cutoffs. |
| **Fake tolerance (tiers)** | 1 | How many bitrate tiers of mismatch are forgiven before a file is flagged FAKE. `0` = no tolerance (any detected shelf below the claimed tier triggers FAKE); `1` (default) = one tier of leeway to absorb normal encoder variance. |
| **Silence gate (dBFS)** | −50 | Segments whose RMS level is below this threshold are skipped (treated as silence). Raise if very quiet passages are causing INCONCLUSIVE results; lower if heavily dynamic material is being gated incorrectly. |
| **Deep (anti-forgery) checks** | `auto` | Controls the stereo/texture second-opinion pass (roughly 2× decode cost per file). `auto` — runs only for claimed-lossless or high-bitrate files. `always` — forces it on every file. `never` — primary spectral detection only (fastest). Changing this setting requires a **Pro** licence. |
| **Worker processes** | all cores | Number of CPU cores to use for parallel analysis. Reduce if the machine becomes unresponsive during large scans. Passed as `--workers N` on the CLI. |

### Clipping settings

| Setting | Default | Description |
|---|---|---|
| **Sample clip threshold (dBFS)** | −0.1 | A run of consecutive samples at/above this level is counted as digital clipping. The default of −0.1 dBFS is intentionally just below true 0 dBFS to allow for minor dither noise. |
| **True peak ceiling (dBTP)** | −1.0 | An inter-sample true peak above this level is flagged as a suspicion (not clipping). The common streaming loudness standard is −1 dBTP. Crossing 0 dBTP (actual over-full-scale) always counts as CLIPPING regardless of this setting. |
| **Per-channel clipping analysis** | on | Judge clipping on each channel independently instead of a mono downmix. Recommended — a single clipped channel can be masked when downmixed to mono. |
| **Full-file clipping pass (slower)** | off | Decode the entire file for the clipping pass rather than only the sampled segments. Catches clips that fall outside the sampled windows; meaningfully slower over large batches. |

### Musical metrics (Elite)

| Setting | Description |
|---|---|
| **BPM / key / energy detection (Elite)** | Enables the BPM, Key, and Energy columns. Requires one additional contiguous decode per file (up to 3 minutes of audio per track), so scans with this on take longer. Requires a **Pro + Elite** licence add-on (separate from the DJ entitlement). |

### Logging

| Setting | Description |
|---|---|
| **Log level** | `DEBUG`/`INFO`/`WARNING`/`ERROR` verbosity written to `specaudit.log`. Use `DEBUG` when diagnosing an unexpected result — it captures every ffmpeg command, segment plan, and resolved binary path. |
| **Log folder** | Where `specaudit.log` is written. **Browse…** picks a new folder; **Open log folder** opens it in your file manager. |
| **Max log file size** | The log file rotates automatically once it reaches this size (default 10 MB). |
| **Rotated backups to keep** | How many older `specaudit.log.N` files are kept alongside the current log (default 3). |

See [§12 Troubleshooting](#12-troubleshooting) for how to use the log to diagnose problems.

### Media cache

SpectralAuditO caches waveform data and spectrogram images to disk so that re-selecting a
previously viewed file is instant — even after restarting the app. The cache lives in
`~/.specaudit/cache/` and grows over time as you view more files.

| Button | What it does |
| --- | --- |
| **Clear waveforms** | Deletes all cached waveform data files (~13 KB each). Waveforms are regenerated the next time you select a file. |
| **Clear spectrograms** | Deletes all cached spectrogram images (~200–500 KB each). Spectrograms are re-rendered the next time you select a file. |

The size label above the buttons shows the current on-disk footprint of each cache
sub-directory. Cache entries for files that no longer exist are never automatically removed;
use these buttons to reclaim disk space when needed.

Cache entries are keyed by file path and modification time, so editing a file automatically
invalidates its entry — you do not need to clear the cache manually after modifying a track.

---

## 10. Command-line reference

```
specaudit [paths...] [options]
```

**Input sources** (at least one required, except for the utility flags below):

| Flag | Description |
|---|---|
| `paths` | One or more files or folders to scan. Folders are scanned recursively unless `--no-recursive` is set. |
| `--from-m3u FILE` | Scan the tracks listed in an M3U/M3U8 playlist. |
| `--from-virtualdj-db FILE` | Scan from a VirtualDJ `database.xml`. |
| `--from-serato-crate DIR` | Scan from a Serato library folder. |
| `--playlist NAME` | (Rekordbox) Restrict to a specific playlist. Repeatable. |
| `--crate NAME` | (Serato) Restrict to a specific crate. Repeatable. |

**Scan options:**

| Flag | Default | Description |
|---|---|---|
| `--no-recursive` | — | Don't recurse into subfolders. |
| `--workers N` | all cores | Number of worker processes. |
| `--deep {auto,always,never}` | `auto` | Stereo/texture second-opinion check depth. |
| `--quiet` | — | Suppress per-file progress lines. |
| `--log-level LEVEL` | `WARNING` | Logging verbosity written to stderr: `DEBUG`, `INFO`, `WARNING`, `ERROR`. Use `DEBUG` to see every ffmpeg command, segment plan, and resolved binary path. |

**Output:**

| Flag | Description |
|---|---|
| `--csv FILE` | Write a CSV report. |
| `--json FILE` | Write a JSON report. |
| `--html FILE` | Write an HTML report with spectrogram thumbnails. |
| `--find-duplicates` | Print a duplicate-track report after scanning. |

**Tag and library write-back:**

| Flag | Description |
|---|---|
| `--write-tags [grouping\|comment]` | Stamp the verdict into each file's own Grouping or Comment tag. Dry-run only unless `--yes` is also passed. |
| `--tag-mode {prefix,append,replace}` | How the stamp combines with existing tag content (default: `prefix`). |
| `--sync-library [grouping\|comment]` | Write the verdict into the source DJ library database. Dry-run only unless `--yes` is also passed. |
| `--yes` | Apply write-back actions instead of just printing a preview. |

**Licence / utility (no scan required):**

| Flag | Description |
|---|---|
| `--machine-id` | Print this machine's licence fingerprint and exit. |
| `--activate KEY` | Activate a licence key online and exit. |
| `--activate-from-file FILE` | Install an offline licence file (`*.sig`) and exit. |

**Exit codes:** `0` = success, `1` = locked feature (Pro required), `2` = Free tier scan cap
reached.

---

## 11. Licensing

Licence requests and general support go to the address shown in **Help → About SpectralAuditO**,
in the offline-activation flow below, and at the bottom of the "What's New / Showcase" page.

### Free tier

The Free tier lets you scan up to **50 files per session**. Report exports, file operations,
tag writing, and DJ library integration are locked.

### Pro licence

Pro removes the scan cap and unlocks all features. A Pro + DJ licence additionally unlocks DJ
library read and write-back. A Pro + Elite licence (a separate add-on entitlement from DJ)
additionally unlocks the opt-in BPM/key/energy musical-metrics detection (§9 Settings).

### Activating online

In the GUI: click the **Licence** button in the toolbar, then **Enter licence key…**

From the CLI:

```sh
specaudit --activate SPEC-XXXX-XXXX-XXXX
```

Requires an internet connection for the initial activation. After that, the licence is verified
locally and offline on every launch.

### Offline / air-gapped activation

If the machine has no internet access:

1. **Get your machine ID** — this is the identifier the licence will be bound to:

   ```sh
   specaudit --machine-id
   ```

   Or in the GUI: click **Licence** in the toolbar, then **Enter licence key…** — the same dialog
   used for online activation now also shows your **Machine ID** under "Offline activation", with
   a **Copy machine ID** button and an **Email issuer…** button that opens a pre-filled email to
   the licence issuer with your machine ID already in the message. (The Machine ID is also shown,
   with a Copy button, on the separate **Help → Licence** info screen.)

2. **Send the machine ID** to your licence issuer — easiest via the **Email issuer…** button above,
   or by copying it into your own message.

3. **Receive a `licence.sig` file** from the issuer and place it at:

   ```
   C:\Users\<YourName>\.specaudit\licence.sig   (Windows)
   ~/.specaudit/licence.sig                      (macOS / Linux)
   ```

   Or install it via the CLI:

   ```sh
   specaudit --activate-from-file licence.sig
   ```

   Or via the GUI: **Help → Licence → Activate from file…**

4. Restart SpectralAuditO. The licence is now active and verified entirely offline.

> **Note:** The licence is bound to this specific machine. If you replace the network card,
> change the hostname, or reinstall the OS, the machine ID will change and you will need to
> request a new `licence.sig` from the issuer.

---

## 12. Troubleshooting

**Windows SmartScreen or Defender warning**
When you first run `SpectralAuditO-Setup.exe` or `specaudit.exe` after downloading, Windows
SmartScreen may show a blue "Windows protected your PC" dialog. This is expected for any
newly released signed application — SmartScreen builds reputation for each release over time.

To proceed: click **More info**, then **Run anyway**.

If you want to verify the signature yourself before running, open PowerShell and run:

```powershell
Get-AuthenticodeSignature "C:\path\to\specaudit.exe" | Select-Object Status, SignerCertificate
```

`Status` should be `Valid`. The `SignerCertificate` subject will show the publisher name. You
can also right-click the file → Properties → Digital Signatures tab to inspect the certificate.

If Defender quarantines the file automatically (rather than just showing a prompt), go to
**Windows Security → Virus & threat protection → Protection history**, find the item, and
choose **Allow**. This is a false positive from heuristic scanning of PyInstaller-packaged
executables; the app contains no malicious code.

---

**"ffmpeg not found" on launch or scan**
SpectralAuditO requires ffmpeg. The standalone Windows build includes it. If you're running from
source, download a static ffmpeg/ffprobe build and place the `.exe` files in `specaudit/bin/`,
or add them to your `PATH`.

**Scan produces no results for a folder**
Check that the folder contains supported formats (`.mp3`, `.flac`, `.m4a`, `.wav`, etc.) and
that **Recursive** is checked if tracks are in subfolders. Video files (`.mp4`, `.mkv`, etc.)
are also supported — only their audio track is analysed.

**All files show INCONCLUSIVE**
This usually means SpectralAuditO could not decode enough audio signal — often because the files
are very short (under ~10 seconds) or nearly silent. Try scanning a full-length track.

**FAKE on a file you believe is genuine**
Hover over the Status cell and read the tooltip. If the fake reason is **Lossy in lossless**
and you ripped the file yourself from a CD, the issue may be that the source CD was pressed
from a lossy master — SpectralAuditO is detecting the lossy encoding baked into the content, not
a problem with your rip.

**High CPU usage during scan**
SpectralAuditO uses all CPU cores by default. Reduce **Worker processes** in Settings (GUI) or
pass `--workers N` on the CLI to limit parallelism.

**Spectrogram doesn't appear after selecting a row**
The spectrogram renders from the full audio file and may take a few seconds for long tracks or
slow storage. If it never appears, the file may be too corrupt to decode fully even if the
short-segment analysis succeeded.

**Diagnosing unexpected results or crashes**
The GUI writes a detailed log to `~/.specaudit/specaudit.log` (created automatically on first
launch, default level DEBUG). This file captures every ffmpeg command issued, segment plans,
resolved binary paths, file errors, and full exception tracebacks from worker processes. Check
this file first when something goes wrong silently.

The log file is rotated automatically once it reaches a size limit (default 10 MB), keeping a
configurable number of older backups (default 3) alongside it as `specaudit.log.1`,
`specaudit.log.2`, etc.

**Settings → Logging** lets you change all of this without editing any files:

| Setting | Effect |
| --- | --- |
| Log level | `DEBUG`/`INFO`/`WARNING`/`ERROR` verbosity written to `specaudit.log`. |
| Log folder | Where `specaudit.log` is written; **Browse…** picks a new folder, **Open log folder** opens it in your file manager. Changing this only affects logging going forward — the previous log file is left where it was. |
| Max log file size | Size (MB) at which the log file rotates. |
| Rotated backups to keep | How many rotated `specaudit.log.N` files are kept. |

Changes apply immediately (no restart needed) and persist across launches in
`~/.specaudit/gui_settings.json`.

On the CLI, use `--log-level DEBUG` to stream the same detail to stderr in real time (the CLI
always logs to stderr only, not to a file, and is unaffected by the GUI's log folder/rotation
settings):

```powershell
specaudit path\to\music --log-level DEBUG 2>debug.log
```

**Licence not recognized after placing `licence.sig`**
- Confirm the file is at the exact path `~/.specaudit/licence.sig` (create the `.specaudit`
  folder if it doesn't exist).
- Confirm the machine ID in the licence matches the one printed by `--machine-id`. If they
  differ, the machine's hardware or hostname changed after the licence was issued — contact the
  issuer for a new licence.
- The file must not be modified after signing; any edit invalidates the signature.
