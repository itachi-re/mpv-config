# Personal mpv Configuration for Linux

<p align="center"><img width=100% src="https://github.com/user-attachments/assets/7d6b7cfd-41c7-4617-b573-933b4b3bd564" alt="mpv screenshot"></p>
<p align="center"><img width=100% src="https://github.com/user-attachments/assets/bfd85e57-f244-4de4-8bf9-750723c51568" alt="mpv screenshot"></p>

## Overview

Personal mpv configuration tuned for **Linux / Wayland / Vulkan**, forked from [Zabooby/mpv-config](https://github.com/Zabooby/mpv-config) and significantly reworked. The original was Windows-only — this fork fixes real bugs present in the upstream config, strips all Windows-specific cruft, and adds proper Wayland/Vulkan/libplacebo settings for mpv 0.41+.

Features tuned profiles (for up/downscaling, live action, anime and HDR content), custom key bindings, the uosc GUI, and a curated set of scripts, shaders and filters. Tested on **openSUSE** with **Hyprland** (Wayland compositor) and an AMD GPU.

> **Before installing**, read this README fully — especially the [What Was Fixed](#what-was-fixed) and [mpv.conf Notes](#mpvconf-notes) sections, as several upstream settings were silently broken.

---

## What Was Fixed

These are real bugs and silent failures present in the upstream config, corrected in this fork.

### `mpv.conf`

**Bugs fixed:**
- `sub-gauss=1.5` **removed** — this option does not exist in mpv. `sub-blur=0.5` (which was already present) is the correct Gaussian blur option for subtitles. `sub-gauss` was silently ignored the whole time.
- `temporal-dither`, `snap-window`, `no-border` — bare flags converted to explicit `=yes` / `border=no` form for clarity and forward compatibility.

**Linux/Wayland enhancements (mpv 0.41+):**
- `gpu-api=vulkan` — mpv 0.41 prefers Vulkan for hardware video decoding on Wayland; making this explicit is the right call with Hyprland and an AMD GPU.
- Added the full libplacebo quality chain: `scale=spline36`, `dscale=mitchell`, `cscale=spline36`, `scale-antiring=0.7`, `sigmoid-upscaling=yes`, `correct-downscaling=yes`, `linear-downscaling=yes`. These were completely absent upstream — the player was running on bare defaults.
- `dither-depth=auto` — prevents banding on 8-bit displays with 10-bit content.
- `video-sync=display-resample` + `tscale=oversample` — required for correct playback timing and for interpolation toggling (`g` key) to actually work.
- `cursor-autohide=1000` — hides the cursor after 1 second of inactivity.

### `profiles.conf`

- Added `deband=yes` to the Deband profiles — they were setting iteration/threshold/range values but never actually **enabling** debanding. The profiles did nothing.
- Removed `sub-hdr-peak` from the HDR profile — this option does not exist in mpv's manual and was silently ignored.
- Added comment on `target-colorspace-hint=yes` — mpv 0.41 adds support for Wayland's color representation protocol for HDR passthrough, so this flag is now actively meaningful on Wayland.
- Added `linear-downscaling=no` to `[4k-Downscaling]` — SSimDownscaler handles this itself; leaving it enabled would interfere.

### `input.conf`

- `vf toggle deblock=filter=weak:block=4` → `vf toggle @deblock:deblock=filter=weak:block=4` — added the `@label:` prefix so the toggle actually works reliably. Without a label, mpv cannot match the filter on subsequent keypresses and the toggle silently breaks after the first press.

### `fonts.conf`

Removed three Windows-only placeholders that had no effect on Linux:
- `<dir>WINDOWSFONTDIR</dir>`
- `<dir>WINDOWSUSERFONTDIR</dir>`
- `<cachedir>LOCAL_APPDATA_FONTCONFIG_CACHE</cachedir>`

Added proper Linux font paths following the XDG standard:
- `/usr/share/fonts`
- `/usr/local/share/fonts`
- `~/.local/share/fonts` (supersedes the deprecated `~/.fonts`)

---

## Scripts and Shaders

- [uosc](https://github.com/darsain/uosc) — Minimalist but highly customisable GUI.
- [evafast](https://github.com/po5/evafast) — Fast-forwarding and seeking on a single key.
- [thumbfast](https://github.com/po5/thumbfast) — High-performance on-the-fly thumbnailer.
- [memo](https://github.com/po5/memo) — Recent files/history menu with uosc integration.
- [InputEvent](https://github.com/natural-harmonia-gropius/input-event) — Enhanced input.conf with conflict-free, low-latency event mechanisms.
- [autodeint](https://github.com/mpv-player/mpv/blob/master/TOOLS/lua/autodeint.lua) — Auto-detects and deinterlaces interlaced content via lavfi's idet filter.
- [autoload](https://github.com/mpv-player/mpv/blob/master/TOOLS/lua/autoload.lua) — Automatically loads playlist entries by scanning the current directory.

---

- [nnedi3 and ravu](https://github.com/bjin/mpv-prescalers) — User shaders for prescaling.
- [FSRCNN](https://github.com/igv/FSRCNN-TensorFlow/releases) — Prescaler based on layered convolutional networks.
- [AniSD ArtCNN](https://github.com/Sirosky/Upscale-Hub/releases/tag/AniSD-ArtCNN) — ArtCNN shader for standard definition anime.
- [Anime4K](https://github.com/bloc97/Anime4K) — Shaders for scaling and enhancing anime. Includes sharpening and upscaling.
- [nlmeans](https://github.com/AN3223/dotfiles/blob/master/.config/mpv/shaders/) — Non-local Means implementation for denoising and adaptive sharpening.
- [Ani4K v2 ArtCNN](https://github.com/Sirosky/Upscale-Hub/releases/tag/Ani4k-v2-ArtCNN) — Targets modern anime (Blu-ray to WEB releases) for 2K/4K upscaling.
- [SSimDownscaler, SSimSuperRes, KrigBilateral, Adaptive Sharpen](https://gist.github.com/igv)
  - **Adaptive Sharpen** — General-purpose sharpening shader.
  - **SSimDownscaler** — Perceptually-based downscaler.
  - **KrigBilateral** — Chroma scaler that uses luma information for high-quality upscaling.
  - **SSimSuperRes** — Corrects ringing and restores sharpness on mpv's built-in upscaler output.

---

## Installation (Linux)

### 1. Install mpv

Use your distro's package manager. **mpv 0.41 or newer is strongly recommended** — several settings in this config (Vulkan video decoding, Wayland color protocol for HDR) require it.

```bash
# openSUSE
sudo zypper install mpv

# Arch / Manjaro
sudo pacman -S mpv

# Debian / Ubuntu (check version — repos may be outdated, consider flatpak or building from source)
sudo apt install mpv

# Fedora
sudo dnf install mpv
```

To check your version:
```bash
mpv --version
```

### 2. Clone this repo

```bash
git clone https://github.com/YOUR_USERNAME/mpv-config.git
```

### 3. Copy config to the mpv config directory

On Linux, mpv reads its config from `~/.config/mpv/` (XDG standard). Copy the contents of `portable_config/` there:

```bash
# Create the config directory if it doesn't exist
mkdir -p ~/.config/mpv

# Copy everything from portable_config into it
cp -r portable_config/. ~/.config/mpv/
```

Or if you prefer to keep it as a live repo and symlink:

```bash
# Symlink individual config files (adjust paths as needed)
ln -sf ~/mpv-config/portable_config/mpv.conf ~/.config/mpv/mpv.conf
ln -sf ~/mpv-config/portable_config/profiles.conf ~/.config/mpv/profiles.conf
ln -sf ~/mpv-config/portable_config/input.conf ~/.config/mpv/input.conf
ln -sf ~/mpv-config/portable_config/fonts.conf ~/.config/mpv/fonts.conf
ln -sf ~/mpv-config/portable_config/scripts ~/.config/mpv/scripts
ln -sf ~/mpv-config/portable_config/shaders ~/.config/mpv/shaders
ln -sf ~/mpv-config/portable_config/script-opts ~/.config/mpv/script-opts
ln -sf ~/mpv-config/portable_config/fonts ~/.config/mpv/fonts
```

### 4. Set paths in script-opts

Open `~/.config/mpv/script-opts/uosc.conf` and set your preferred default directory for the uosc file browser:

```
default_directory=/your/media/path
```

Open `~/.config/mpv/script-opts/memo.conf` and verify the history log path points somewhere writable, e.g.:

```
history_path=~/.config/mpv/script-opts/memo-history.log
```

### 5. Adjust settings to your system

Review `mpv.conf` and `profiles.conf` and adjust to your hardware. Key things to check:

- `hwdec=auto` — this is a safe default. If you have specific hardware (e.g. AMD with VAAPI), you may want `hwdec=vaapi` or `hwdec=vulkan` explicitly.
- `gpu-api=vulkan` — works well on AMD/Intel with Mesa. If you have issues, try `gpu-api=auto`.
- `vo=gpu-next` — required for libplacebo and most shader features on mpv 0.41.
- Shader profiles in `profiles.conf` — some shaders are GPU-intensive. Adjust or disable per profile based on your hardware.

Refer to the [mpv manual](https://mpv.io/manual/master/) for what each option does.

### 6. Optional: yt-dlp for streaming

To stream from YouTube and other sites, install yt-dlp:

```bash
# Via pip
pip install yt-dlp

# Or via your package manager
sudo zypper install yt-dlp   # openSUSE
sudo pacman -S yt-dlp        # Arch
```

### 7. Verify

Launch mpv and open the console with `` ` `` (backtick). Check for any warnings or errors. Common issues and their fixes are listed below.

---

## mpv.conf Notes

This config sets `vo=gpu-next`, which uses the **libplacebo** rendering backend. This is required for the shader profiles, correct HDR handling, and the libplacebo scaler chain. If you see errors about `gpu-next` not being available, your mpv build may be missing libplacebo — check your distro's package or build from source with libplacebo enabled.

`gpu-api=vulkan` is set explicitly. If Vulkan causes issues on your system (unlikely on AMD/Intel with Mesa, possible on older NVIDIA), change it to `gpu-api=auto` as a fallback.

`video-sync=display-resample` is required for `tscale=oversample` (motion interpolation) to work. Without it, toggling interpolation with `g` has no effect.

---

## Troubleshooting

**Subtitles look wrong / blur not applying**
Ensure `sub-blur=0.5` is set in `mpv.conf`. The old `sub-gauss` option doesn't exist in mpv and does nothing.

**Deband profiles not working**
The upstream config had this bug. This fork fixes it — deband profiles now include `deband=yes`. If you copied over an old `profiles.conf`, replace it with the one from this repo.

**`g` key (interpolation toggle) does nothing**
You need `video-sync=display-resample` and `tscale=oversample` in `mpv.conf`. Both are set in this config.

**Deblock toggle breaks after first press**
Fixed in this fork's `input.conf` via the `@deblock` label. If using an old `input.conf`, update the deblock line.

**Font rendering issues / wrong fonts**
The `fonts.conf` in this repo points to standard Linux font paths. Ensure your fonts are installed in `/usr/share/fonts`, `/usr/local/share/fonts`, or `~/.local/share/fonts`. The old `~/.fonts` path is deprecated and not included here.

**HDR content looks washed out**
Check that `target-colorspace-hint=yes` is set in the HDR profile (it is in this config). On Wayland with mpv 0.41+, this enables the color representation protocol for proper HDR passthrough to your compositor.

**Black screen / Vulkan errors**
Try changing `gpu-api=vulkan` to `gpu-api=auto` in `mpv.conf`. Also ensure your Mesa or GPU drivers are up to date.

**Console shows "No such option: sub-hdr-peak"**
This was a bug in the upstream `profiles.conf`. It's removed in this fork. If you see it, you have an old version of the file.

---

## Key Bindings

Custom key bindings are in `input.conf`. Most player functions are also accessible through the uosc menu (right-click) and the controls above the timeline. Refer to the [mpv manual](https://mpv.io/manual/master/) and [uosc commands](https://github.com/tomasklaen/uosc#commands) for customisation.

---

## File Structure

```
mpv-config/
│
├── portable_config/                  # Drop contents into ~/.config/mpv/
│   │
│   ├── fonts/
│   │   ├── ClearSans-Bold.ttf
│   │   ├── JetBrainsMono-Regular.ttf
│   │   ├── uosc_icons.otf
│   │   └── uosc_textures.ttf
│   │
│   ├── script-opts/                  # Per-script configuration files
│   │   ├── console.conf
│   │   ├── evafast.conf
│   │   ├── memo.conf
│   │   ├── memo-history.log          # Created automatically on first run
│   │   ├── thumbfast.conf
│   │   └── uosc.conf                 # Set your default media directory here
│   │
│   ├── scripts/
│   │   ├── uosc/                     # GUI (full folder)
│   │   ├── autodeint.lua
│   │   ├── autoload.lua
│   │   ├── evafast.lua               # Hold right arrow to activate
│   │   ├── inputevent.lua
│   │   ├── memo.lua
│   │   └── thumbfast.lua
│   │
│   ├── shaders/                      # GLSL upscaling/processing shaders
│   │   ├── A4K_Clamp_Highlights.glsl
│   │   ├── A4K_Restore_CNN_Soft_M.glsl
│   │   ├── adasharp_luma.glsl
│   │   ├── Ani4Kv2_ArtCNN_C4F32_i2.glsl
│   │   ├── AniSD_ArtCNN_C4F32_i4.glsl
│   │   ├── F8.glsl
│   │   ├── F16.glsl
│   │   ├── krigbl.glsl
│   │   ├── nlmeans_luma.glsl
│   │   ├── nnedi3_nns32_win8x4.glsl
│   │   ├── nnedi3_nns64_win8x4.glsl
│   │   ├── ravu_z_ar_r3.glsl
│   │   ├── ssimds.glsl
│   │   └── ssimsr.glsl
│   │
│   ├── fonts.conf                    # Linux font paths (XDG standard)
│   ├── input.conf                    # Key bindings + uosc menu customisation
│   ├── mpv.conf                      # Main config (Vulkan/Wayland/libplacebo tuned)
│   └── profiles.conf                 # Conditional profiles (anime, HDR, 4K, deband, etc.)
│
└── README.md
```

---

## Useful Links

- [mpv wiki](https://github.com/mpv-player/mpv/wiki) — Official wiki with scripts, shaders, FAQs and discussions.
  - [Awesome mpv](https://github.com/stax76/awesome-mpv) — Curated list of user resources in distinct sections.
- [mpv manual](https://mpv.io/manual/master/) — Complete reference for all options and scripting APIs.
- [libplacebo documentation](https://libplacebo.org/) — Reference for the rendering backend used by `vo=gpu-next`.
- [Original upstream config](https://github.com/Zabooby/mpv-config) — Windows-focused config this fork is based on.
