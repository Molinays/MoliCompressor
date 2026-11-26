# MoliCompressor — User guide

MoliCompressor compresses videos and photos in batch, one file at a time, keeping the folder tree of
the source. This guide explains every screen and every setting.

## 1. Importing files

Drag files or whole folders onto the drop area, or use **Choose files** / **Choose folder**. Folders
are scanned recursively; the queue keeps the relative path, so `Holiday/2024/clip.mov` becomes
`<output>/Holiday/2024/clip.mp4`.

Duplicates are skipped: adding the same file twice does not enqueue it again.

**Output** defaults to `Videos/MoliCompressor` and can be changed at any time; the `↗` button opens
it in the file manager. Output files never overwrite their source: if the target path would be the
input file, the name gets a `-compressed` suffix.

## 2. Presets

Presets are starting points, not locked modes. Selecting one updates the subtitle in the header
(`Max Compression · HEVC · GPU · CRF 28 · 1080p · MP4`) and the predicted size of every queued file.

| Preset | Best for |
| --- | --- |
| **Max Compression** | Smallest files that still look good on a screen (up to −80%). |
| **Extreme** | Upload archives: 900p at 30 fps, tiny. |
| **Tiny 480p** | Previews, sharing, very long archives. |
| **Web 720p** | Embedding in a website. |
| **Balanced** | Halve the size with near-original quality; keeps all audio tracks. |
| **High Quality** | Near-transparent HEVC in MKV with subtitles and every audio track. |
| **Master 4K** | Archive master, full resolution, CRF 16. |
| **Social 1080p** | H.264 for WhatsApp, Instagram, TVs. |
| **Chat & Discord** | Small H.264 720p that plays anywhere. |
| **Gaming 60** | High motion at 60 fps without huge files. |
| **Fast GPU** | Seconds per minute of video, larger files. |

Changing a preset while the queue is running is blocked: settings are frozen at start, like in
HandBrake.

## 3. Engine and CPU load

- **Max GPU** — jobs are encoded by the hardware encoder of the same family as the preset
  (HEVC → `hevc_nvenc`/`hevc_qsv`/`hevc_amf`, H.264 → `h264_*`, AV1 → `av1_*`). Decoding uses the
  GPU too. If a particular file cannot be hardware-decoded, it is retried in software automatically.
- **CPU only** — always x265/x264.
- **Balanced** — leaves two cores free; **Max speed** uses all of them.

## 4. Advanced settings

Each field shows the preset value; editing one marks the panel **Custom overrides** and the header
gains `· custom`. **Reset** returns to the preset. Overrides apply to the whole queue when you press
Compress.

**Container / Video codec** — MP4, MKV, WEBM; x265, x264, VP9 or any detected hardware encoder
(`auto-hardware` follows *Max GPU*). Choosing WEBM with an H.26x encoder switches to VP9 so the file
stays playable.

**Quality (CRF / CQ)** — lower is better. 18–22 is visually near-lossless, 24–28 is the sweet spot,
30+ is for size. With hardware encoders this is CQ.

**Encoder speed** — x265/x264 presets (`slow`…`placebo`) or NVENC presets (`p1` fast … `p7` slow).
Slower means smaller files at the same quality.

**Resolution / Frame rate** — only downscale, never upscale. Frame rate caps at 60/30/25/24.

### Audio

- **Audio** — AAC, Opus, *Copy* (no re-encode, keeps the original codec) or *None*.
- **Audio bitrate / Sample rate / Channels** — hidden for *Copy* and *None*, since they do not apply.
- **Tracks to export**
  - *Primary only* — the track flagged as default in the file (usually the right language).
  - *All tracks* — every audio track, remuxed into one output file.
  - *Merge all into one* — all tracks are mixed into a single stereo track (`amix`), useful when
    different tracks carry different audio rather than different languages.
  - *Custom selection* — only the track numbers you list, 1 = first audio track in the file.
- **Track numbers** — visible for *Custom selection*, e.g. `1,3`. Invalid numbers are ignored.

### Subtitles and container extras

- **Subtitles** — *Auto* keeps them only in MKV, *Keep all* forces them (converted to `mov_text` in
  MP4 and `webvtt` in WEBM), *Remove* drops them.
- **Metadata** — *Keep* copies tags and chapters, *Strip* writes none.
- **Web fast start** — MP4 only; moves the index to the beginning so playback starts instantly.
- **Deinterlace** — Yadif, for old camcorder/DVD sources.
- **Light denoise** — `hqdn3d`, removes grain and compression bands before encoding.
- **Keyframe every** — GOP size in frames, 0 = encoder default.
- **Max bitrate** — cap in kbps with a 2× buffer, 0 = unlimited.
- **Encoder parameters** — raw `key=value:key=value` appended to the preset tuning. They belong to
  the current encoder: x265 parameters are dropped automatically when you switch to x264 or to
  hardware.

## 5. Queue

The Queue tab shows time remaining for the whole job, files done/total, size before, **predicted**
size, size after and saved percentage.

Each row has before → predicted (or before → after), a progress bar, per-file ETA and a status of
Queued / Running / Done / Error / Canceled.

- **×** cancels that single file. If it is running, the encode is killed and the queue continues with
  the next one.
- **Cancel all** stops the queue after killing the current encode.
- **Retry** puts failed and canceled files back in the queue.
- **Clear** empties the queue (disabled while running).
- **↗** opens the folder containing a completed output video and selects the compressed file.
- Double-click a finished row to reveal the output file.

### Predicted size

The prediction is computed from the source bitrate (a proxy for how complex the content is), scaled
by the CRF curve (−1 stop every 6 points), the resolution and frame-rate reduction, the codec and
the audio settings. It is recalculated when you change preset or overrides. Expect **±15–25%** on real
footage: a CRF target controls quality, not a fixed bitrate.

## 6. Console

**Console** (in Compress and in the Queue) opens the FFmpeg log: the full command line, warnings,
errors, and the outcome of each file. Lines starting with `✖` are failures, `!` a GPU-decode retry,
`✔` a completed encode. **Copy** copies everything, **Clear** empties the session buffer (4000 lines
are kept).

## 7. Photos

JPEG, PNG and WebP are resized to **Photo max edge** (preserving the aspect ratio) and encoded with
**Photo quality** 1–12 (1 = almost original, 12 = very compressed). EXIF orientation is preserved.

## 8. Info

- **Graphics** — GPU name, driver, VRAM and the hardware encoders that answered a real encode test.
  **Re-scan** repeats that test.
- **System** — app version, OS, architecture, CPU, cores, RAM, FFmpeg version and paths, data folder.
- **Dependencies** — FFmpeg, FFprobe, WebView2 (Windows) and the app data folder. Missing FFmpeg can
  be installed into the user profile with one click, no administrator rights.
- **Updates** — current version, available release notes, download progress, install button.

The page is prefetched at startup, so it opens complete without pressing anything.

## 9. Updates

At startup (3.5 s after the window appears) the app asks GitHub for the latest release. If a newer
version exists, a chip appears in the header and the Info tab offers the download. Nothing is
installed without a click. Downloads are verified with a minisign signature embedded in the app.

On Windows the NSIS installer closes the app, updates it and starts it again. On Linux the AppImage
is replaced in place.

## 10. Command line

```
molicompressor --scan <path>
molicompressor --presets
molicompressor --estimate <input> [--preset id] [--crf n] [--gpu force|cpu]
molicompressor --compress <input> <output> [--preset id] [--threads n] [--gpu force|cpu]
               [--crf n] [--container mp4|mkv|webm] [--encoder libx265|libx264]
               [--speed slower] [--resolution 1080p] [--fps 30]
               [--audio aac|opus|copy|none] [--audio-bitrate 128]
               [--audio-tracks first|all|merge|custom] [--audio-select 1,3]
               [--subtitles auto|keep|none] [--params k=v:k=v]
```

`--estimate` prints JSON with the predicted bytes and the encoder that would be used.

## Troubleshooting

| Symptom | What to do |
| --- | --- |
| A file fails with `Invalid data found when processing input` | The source is truncated or DRM-protected. Open the Console for the exact FFmpeg message. |
| Saved percentage is negative | The preset re-encodes at a higher quality than the source (e.g. Master 4K on a small clip). Use a smaller preset. |
| `No hardware encoder` in the GPU badge | The driver does not answer the encoder test. Install/update the GPU driver, or use CPU only. |
| AppImage does not start | It needs FUSE: install `libfuse2` on Ubuntu 22.04+, or run with `--appimage-extract-and-run`. |
| Update check fails | No release published yet, or the network blocks GitHub. |
