# Fixing Missing FFmpeg Text Filters on macOS

## Problem

Homebrew's core FFmpeg formula was slimmed down in 2026, with the following comment in the formula:

> Only add dependencies required for dependents in homebrew-core.

As a result, Homebrew's FFmpeg build no longer includes `libass` or `libfreetype` by default.

The installed version, **FFmpeg 9.0.2_1**, is missing three important filters:

- `ass` — Required for rendering ASS/SSA captions.
- `subtitles` — Required for rendering subtitle files.
- `drawtext` — Required for rendering text overlays, including the hook overlay.

Both caption rendering and hook overlay generation failed because these filters were unavailable.

**Impact:** The video rendered successfully, but without captions or text overlays.

**Root cause:** Missing FFmpeg build dependencies, not a bug in the application code.

## Solution

Install a full-featured FFmpeg build that includes the required text-rendering filters.

### Option 1: Download a Precompiled FFmpeg Build (Recommended)

**Platform:** Intel Mac (x86_64)  
**Advantage:** No compilation required; faster installation.

1. Create a local binary directory:

   ```bash
   mkdir -p ~/bin && cd ~/bin
   ```

2. Download and extract FFmpeg:

   ```bash
   curl -L -o ffmpeg.zip https://evermeet.cx/ffmpeg/getrelease/zip
   unzip -o ffmpeg.zip
   rm ffmpeg.zip
   ```

3. Download and extract FFprobe:

   ```bash
   curl -L -o ffprobe.zip https://evermeet.cx/ffmpeg/getrelease/ffprobe/zip
   unzip -o ffprobe.zip
   rm ffprobe.zip
   ```

4. Prioritize the downloaded binaries in your `PATH`:

   ```bash
   export PATH="$HOME/bin:$PATH"
   ```

   Add this line to `~/.zshrc` to make the change persistent.

5. Verify the required filters are available:

   ```bash
   ffmpeg -hide_banner -filters | grep -E " (ass|subtitles|drawtext) "
   ```

   All three filters should appear in the output.

### Option 2: Install FFmpeg Through a Homebrew Tap

**Platform:** macOS  
**Disadvantage:** Compiles from source and can be slow on Intel Macs.

1. Uninstall the existing FFmpeg package:

   ```bash
   brew uninstall ffmpeg
   ```

2. Add the alternative FFmpeg tap:

   ```bash
   brew tap homebrew-ffmpeg/ffmpeg
   ```

3. Install FFmpeg with `libass` support:

   ```bash
   brew install homebrew-ffmpeg/ffmpeg/ffmpeg --with-libass
   ```

4. Verify the required filters:

   ```bash
   ffmpeg -hide_banner -filters | grep -E " (ass|subtitles|drawtext) "
   ```

   **Note:** Confirm that the tap's build options enable all three filters. The `--with-libass` option alone does not guarantee `drawtext` support.

## Recommended Approach

**Option 1 is preferred** because it avoids the lengthy source compilation process on Intel Macs.

Once FFmpeg is installed and the required filters are verified, rerun the video generation process.

The application already caches:

- The generated script.
- The generated images.

Consequently, the next run should reuse those assets and skip the approximately **17-minute image generation step**.

## Outstanding Verification

The following items have not yet been verified:

- Whether the Evermeet FFmpeg and FFprobe download URLs remain operational.
- Whether the downloaded FFmpeg build includes all required filters.
- Whether `hook_overlay` rebuilds successfully without additional errors.

## Summary

| Item | Details |
|---|---|
| Affected version | FFmpeg 9.0.2_1 |
| Installation source | Homebrew core |
| Missing filters | `ass`, `subtitles`, `drawtext` |
| Root cause | FFmpeg compiled without required text-rendering dependencies |
| Application bug | No |
| Video rendering | Successful, but without text |
| Recommended fix | Precompiled full-featured FFmpeg build |
| Expected recovery | Rerun generation using cached assets |
| Verification status | Pending |

**Resolution status:** Fix identified; installation and successful regeneration not yet confirmed.

<br>
