# Hashbrown

Hashbrown is a Windows video-editing toolkit built for legal and investigative professionals. It bundles three tools under one launcher:

- **Redact** — mute one or more audio segments of a video (optionally with a visual mute-icon overlay).
- **Amplify** — boost or reduce volume of audio/video files, with a 20-second audio preview.
- **Trim** — cut a video to a start timestamp, an end timestamp, or both.

All three tools share a common scaffolding (drag-and-drop, ffmpeg discovery, progress reporting, cancel support) so the suite is easy to extend with new tools.

## Project layout

```
hashbrown/
├── launcher.py          # Single entry that shows three tool buttons
├── shared/
│   ├── app_base.py      # BaseToolApp window chrome
│   ├── ffmpeg_util.py   # ffmpeg discovery, probe, progress runner
│   ├── platform_util.py # Windows subprocess hiding
│   └── time_field.py    # HH:MM:SS auto-navigating input widget
└── tools/
    ├── redact.py        # Audio redaction (formerly Hashbrown.py)
    ├── amplify.py       # Volume amplification (formerly Grits)
    └── trim.py          # Video trim
assets/                  # logo.png, mute.png, mute_2.png
Hashbrown.spec           # PyInstaller spec — builds a single Hashbrown.exe
```

## Running from source

```
pip install -r requirements.txt
python -m hashbrown.launcher
```

You can also run individual tools directly:

```
python -m hashbrown.tools.redact
python -m hashbrown.tools.amplify
python -m hashbrown.tools.trim
```

## Usage

### Redact

1. Drag a video onto the window or click **Browse**.
2. Enter **Start** and **End** times (HH:MM:SS) for the segment to censor.
3. Click **+ Additional Segment** for more.
4. Toggle **Include Overlay Image** to either:
    - **Unchecked (Fast Mode):** mute audio only, copy video stream (very fast, no re-encode).
    - **Checked (Standard Mode):** mute audio AND burn a mute icon onto the video during the redacted segments (requires re-encoding).
5. Click **Process Video**. Output saves alongside the source with `_redacted` or `_redacted_fast` suffix.

### Amplify

1. Load a video or audio file (mp4/mov/mkv/mp3/wav/flac/etc.).
2. Pick a preset (×0.5, ×2.0, ×5.0, ×10) or open **Advanced Options** for a custom value (0.01–999.99).
3. Click **Original Audio Preview** or **Amplified Audio Preview** to listen to a 20-second sample. Press **ESC** to stop.
4. Click **Process Media**. Output saves with a `_vol{N}` suffix (e.g. `clip_vol2.00.mp4`).

### Trim

1. Load a video.
2. Tick **Set start time** and/or **Set end time** and fill in the timestamps.
3. (Optional) Tick **Re-encode** if stream-copy at the cut points causes A/V sync issues. Stream-copy is the default and is fast but cuts on the nearest keyframe.
4. Click **Trim Video**. Output saves with a `_trimmed` suffix.

## Internal operations

All tools shell out to FFmpeg (bundled via `imageio_ffmpeg`). Redact additionally probes for NVIDIA NVENC hardware acceleration when the overlay mode is enabled and writes `encoder.log` next to the exe for diagnosis.

### FFmpeg command examples

**Redact, Fast Mode (audio mute only)**

```bash
ffmpeg -y -i input.mp4 -c:v copy -af "volume=enable='between(t,10,20)+between(t,45,50)':volume=0" -c:a aac -b:a 128k input_redacted_fast.mp4
```

**Redact, Overlay Mode**

```bash
ffmpeg -y -i input.mp4 -i mute_icon.png -filter_complex "[0:v][1:v]overlay=5:5:enable='between(t,10,20)'[v_out];[0:a]volume=enable='between(t,10,20)':volume=0[a_out]" -map [v_out] -map [a_out] -c:v libx264 -preset fast -b:v 2M -c:a aac -b:a 128k input_redacted.mp4
```

**Amplify**

```bash
ffmpeg -y -i input.mp4 -c:v copy -af "volume=2.0" input_vol2.00.mp4
```

**Trim (stream copy)**

```bash
ffmpeg -y -ss 00:00:30 -to 00:02:00 -i input.mp4 -c copy input_trimmed.mp4
```
