# Building MoliCompressor

The frontend is plain HTML/CSS/JS bundled by Tauri; the backend is Rust. There is no JS build step.

## Requirements

- **Rust** stable (Windows: the `x86_64-pc-windows-gnu` toolchain works, MSVC is not required).
- **Node 20+** for the Tauri CLI (`npm install` in this folder).
- **FFmpeg + FFprobe** in `resources/` — they are not in the repository (about 157 MB).

### Windows

```powershell
cd src-tauri
cargo build            # debug
cargo build --release  # release, LTO + opt-level=s
```

FFmpeg/FFprobe: download `ffmpeg-release-essentials.zip` from
[gyan.dev](https://www.gyan.dev/ffmpeg/builds/) and copy `bin/ffmpeg.exe` and `bin/ffprobe.exe`
into `resources/`.

### Linux

```bash
sudo apt install libwebkit2gtk-4.1-dev libgtk-3-dev libayatana-appindicator3-dev librsvg2-dev patchelf
cargo build --release
```

Download a static FFmpeg build
([johnvansickle](https://johnvansickle.com/ffmpeg/) works) and place `ffmpeg` and `ffprobe` in
`resources/`. The bundle definition in `src-tauri/tauri.conf.json` points at `ffmpeg.exe` /
`ffprobe.exe`; for a Linux package replace that map with:

```json
"resources": {
  "../resources/ffmpeg": "ffmpeg",
  "../resources/ffprobe": "ffprobe"
}
```

The release workflow does exactly this before building on Ubuntu.

## Bundles

```bash
# Windows: signed NSIS installer
npx @tauri-apps/cli build --bundles nsis

# Linux: AppImage (the format the updater can replace)
npx @tauri-apps/cli build --bundles appimage

# development window
npx @tauri-apps/cli dev
```

Artifacts land in `src-tauri/target/release/bundle/<format>/`; the executable is
`src-tauri/target/release/molicompressor` (`molicompressor.exe` on Windows).

The release workflow also creates a single-file portable Windows executable. It packages
`MoliCompressor.exe`, `ffmpeg.exe` and `ffprobe.exe` into a self-extracting `.exe` that launches the
app without installing it.

## Signed builds (required for the updater)

`bundle.createUpdaterArtifacts` is `true`, so a build fails without a signing key:

```bash
export TAURI_SIGNING_PRIVATE_KEY="$PWD/.tauri/molicompressor.key"
export TAURI_SIGNING_PRIVATE_KEY_PASSWORD=""
npx @tauri-apps/cli build --bundles nsis
```

This produces `*-setup.exe.sig` next to the installer. The matching public key is embedded in
`src-tauri/tauri.conf.json` under `plugins.updater.pubkey`. Losing the private key means installed
copies will reject every future update.

## Continuous integration

`.github/workflows/release.yml` runs on every `v*` tag:

1. **build** (matrix: `windows-latest` → NSIS, `ubuntu-22.04` → AppImage) downloads FFmpeg, patches
   the Linux resource names, builds, signs and uploads the bundles to the release.
2. **manifest** runs once, downloads the `.sig` files and writes a single `latest.json` with both
   `windows-x86_64` and `linux-x86_64` entries. One writer avoids the race where the last matrix job
   overwrites the other platform's entry.

`scripts/latest-json.cjs` can be run locally against a folder of signatures:

```bash
node scripts/latest-json.cjs --version 1.0.2 --tag v1.0.2 --repo Molinays/MoliCompressor --sigs sigs
```

## Site

`site/index.html` is standalone: open it directly, or serve the folder
(`node build/shots/serve.cjs` → `http://127.0.0.1:5178/site/index.html`). It uses no external
fonts or scripts, so it also works offline. `.github/workflows/pages.yml` publishes it to GitHub
Pages once Pages is set to "GitHub Actions" in the repository settings.

## Documentation screenshots

`build/shots/` renders the real UI with a stubbed Tauri bridge (`mock.js`) and sample data, so the
screenshots match the app without touching a real queue:

```bash
node build/shots/serve.cjs   # http://127.0.0.1:5178/build/shots/index.html
```

Views can be selected from the URL (`?view=queue`, `?view=info`) and the console with
`?view=queue&console=1`. Regenerate the PNGs with a headless browser, for example:

```bash
chrome --headless=new --window-size=1240,880 \
  --screenshot="docs/images/queue.png" \
  "http://127.0.0.1:5178/build/shots/index.html?view=queue"
```

## Project layout

```
src-tauri/src/engine.rs    FFmpeg argument building, probing, hardware detection, CLI
src-tauri/src/queue.rs     queue worker, add/start/cancel
src-tauri/src/presets.rs   presets and user overrides
src-tauri/src/deps.rs      system info, dependency check, FFmpeg installer
src-tauri/src/state.rs     shared state, log buffer, snapshots
ui/                        HTML/CSS/JS frontend (no bundler)
site/                      marketing site (static, published by pages.yml)
build/shots/               screenshot harness (not shipped)
scripts/latest-json.cjs    updater manifest generator
.github/workflows/         release pipeline
```
