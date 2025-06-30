# FFmpeg audio converter and downsampler using Automator for macOS
Useful automator-applications for macOS that allows you to use FFmpeg to convert WAV/FLAC to AIFF, M4A to MP3, do bitrate downsampling, do audio normalization. Perfect for DJs to prepare and convert their audio files.

# Installation
Make sure that you have installed Homebrew for these applications to work. Click [here](https://brew.sh/) for installation instructions.

As long as you have installed Homebrew then the applications should automatically install the required dependencies. If the applications are still not working then it is recommended to open the terminal and install the dependencies manually by running the following command:

``brew install ffmpeg``

``brew install trash``

# Usage
1. Copy the applications to your Applications-folder
2. Optionally drag the applications to your dock for easy access
3. Find the file(s) you want to convert or downsample - drag them and drop them on top of one of the applications, depending on if you want to convert or downsample the files.

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