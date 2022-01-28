# FFmpeg audio converter and downsampler using Automator for macOS
Two automator-applications for macOS that allows you to use FFmpeg to convert WAV/FLAC to AIFF, M4A to MP3, and do bitrate downsampling. Perfect for DJs to prepare and convert their file.

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
