# Grits

Grits is a media amplification tool that allows users to easily adjust the volume of video or audio files. It provides preset amplification options and supports custom values for precise control.

## Features

- **Load Any Media:** Drag and drop or browse to select video (MP4, AVI, MOV, MKV, FLV, WMV) or audio files (MP3, WAV, AAC, M4A, OGG, FLAC)
- **Preset Amplification:** Four quick preset buttons for common amplification levels:
  - ×0.5 (reduce volume by half)
  - ×2.0 (double the volume)
  - ×5.0 (5x amplification)
  - ×10 (10x amplification)
- **Advanced Options:** Custom amplification from 0.01 to 999.99 for precise control
- **Preview Function:** Listen to a random 10-second segment from the center of your media to test the amplification before processing
- **Fast Processing:** Uses FFmpeg with video stream copy (no re-encoding) for quick processing

## Usage

1. **Load Media:** Run the executable and drag and drop a video or audio file into the window, or click "Browse" to select a file.
2. **Select Amplification:** Click one of the preset buttons (×0.5, ×2.0, ×5.0, ×10) or check "Advanced Options" to enter a custom value.
3. **Preview (Optional):** Click "Preview (10s Sample)" to hear a random 10-second segment with the selected amplification applied.
4. **Process:** Click "Process Media". The output file will be saved in the same directory as the source file with a suffix indicating the amplification level (e.g., `_vol2.00.mp4`).

## Internal Operations

Grits utilizes FFmpeg for all media processing. It copies the video stream directly without re-encoding for maximum speed and quality preservation, while only processing the audio to apply the volume filter.

### FFmpeg Command Example

```bash
ffmpeg -i input.mp4 -c:v copy -af "volume=2.0" output_vol2.00.mp4
```

- `-c:v copy`: Copies video stream without re-encoding (preserves original quality)
- `-af "volume=2.0"`: Applies audio filter to amplify volume by 2x
- Output is saved with the amplification value in the filename

## System Requirements

- FFmpeg (bundled with the application via imageio_ffmpeg)
- Windows, macOS, or Linux

## Building from Source

1. Install requirements: `pip install -r requirements.txt`
2. Run directly: `python Grits.py`
3. Build executable: `pyinstaller Grits.spec`
