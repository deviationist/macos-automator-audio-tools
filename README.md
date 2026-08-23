# FFmpeg audio converter and downsampler using Automator for macOS
Useful automator-applications for macOS that allows you to use FFmpeg to convert WAV/FLAC/ALAC to AIFF, do bitrate downsampling, do audio normalization. Perfect for DJs to prepare and convert their audio files.

# Applications

## FFmpeg Audio Converter
Converts lossless files to AIFF. `.m4a` can hold either lossy AAC or lossless ALAC, so the audio codec - not the file extension - decides what happens:

| Input | Audio codec | Output |
| --- | --- | --- |
| `.flac` | - | `.aiff` |
| `.wav` | - | `.aiff` |
| `.m4a` | ALAC (lossless) | `.aiff` |
| `.m4a` | AAC (lossy) | left untouched |

**Lossy files are deliberately left alone.** Converting AAC to MP3 is a lossy-to-lossy transcode: it discards detail the source still had and produces a larger file, with nothing gained. If a player can read the `.m4a`, keeping the original AAC is always the better copy.

Embedded cover art and tags are carried over. AIFF output keeps the bit depth of the source, so a 24-bit FLAC or ALAC stays 24-bit.

The app asks once whether to move the source files to the trash, and only does so for files that converted successfully. M4A files whose codec cannot be read (a truncated or corrupt file, or a missing `ffprobe`) are left untouched and listed in a summary dialog at the end, since it is not possible to tell whether they were lossless.

If two dropped files would produce the same output name - `song.flac` and `song.wav` both want `song.aiff` - the second one is written as `song-2.aiff` instead of overwriting the first.

## FFmpeg Audio Rate Downsampler
Asks for a maximum sample rate (default 48000) and re-encodes each dropped file to it, keeping the original format. Output is written alongside the source as `<name>-downsampled.<ext>`.

## FFmpeg Audio Normalizer
Applies FFmpeg's `loudnorm` filter to each dropped file, keeping the original format. Output is written alongside the source as `<name>-normalized.<ext>`.

## Audio File Evaluator
Scans dropped MP3 files (and recurses into dropped folders) with `mp3val` to find corrupt files, then offers to move the ones you select to the trash. MP3 only.

# File extensions
All applications match file extensions case-insensitively, so `.MP3`, `.WAV`, `.FLAC` and `.M4A` are handled exactly like their lowercase counterparts.

# Installation
Make sure that you have installed Homebrew for these applications to work. Click [here](https://brew.sh/) for installation instructions.

As long as you have installed Homebrew then the applications should automatically install the required dependencies. If the applications are still not working then it is recommended to open the terminal and install the dependencies manually by running the following command:

``brew install ffmpeg``

``brew install trash``

``brew install mp3val``

FFmpeg provides both `ffmpeg` and `ffprobe`; the converter needs both.

# Usage
1. Copy the applications to your Applications-folder
2. Optionally drag the applications to your dock for easy access
3. Find the file(s) you want to process - drag them and drop them on top of the application you want to use (see [Applications](#applications) above).

# Debugging
If something is not working then you can open the Automator-application, open the convert/downsample application, run the process from there, and then inspect the various output during the process.

# Applying icon

The icon should be embedded in the app-files, but if not here is how to build and apply them. First get a base icon in PNG-format, preferably minimum 512x512. Then run the commands below:

```mkdir MyIcon.iconset
sips -z 16 16     Icon1024.png --out MyIcon.iconset/icon_16x16.png
sips -z 32 32     Icon1024.png --out MyIcon.iconset/icon_16x16@2x.png
sips -z 32 32     Icon1024.png --out MyIcon.iconset/icon_32x32.png
sips -z 64 64     Icon1024.png --out MyIcon.iconset/icon_32x32@2x.png
sips -z 128 128   Icon1024.png --out MyIcon.iconset/icon_128x128.png
sips -z 256 256   Icon1024.png --out MyIcon.iconset/icon_128x128@2x.png
sips -z 256 256   Icon1024.png --out MyIcon.iconset/icon_256x256.png
sips -z 512 512   Icon1024.png --out MyIcon.iconset/icon_256x256@2x.png
sips -z 512 512   Icon1024.png --out MyIcon.iconset/icon_512x512.png
cp Icon1024.png MyIcon.iconset/icon_512x512@2x.png
iconutil -c icns MyIcon.iconset
rm -R MyIcon.iconset```

Find the app-file you want to apply the icon to, right click and click "Get Info". Then drag-n-drop the icon file to the upper left corner on top of the current icon. In some cases you can also just drag-n-drop the PNG instead of creating the iconset. Et voila!