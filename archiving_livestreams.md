# Archiving Livestreams

This guide covers how to record livestreams as they are occuring. This is useful for streams that will not be archived later.

## Table of Contents

1. [Recording Regular Streams](#recording-regular-streams)
    * [Prerequisites](#prerequisites)
    * [Instructions](#instructions)
    * [Advanced Scenarios](#advanced-scenarios)
2. [Recording Members Only Streams](#recording-members-only-streams)
3. [Post Processing](#post-processsing)
4. [FAQ](#faq)

## Recording Regular Streams
This is to record publicly availble streams using [Streamlink](https://streamlink.github.io/). Youtube-dl works but Streamlink offers a better user experience for this scenario (it is common for people to corrupt their recordings with youtube-dl). The following steps will record a stream while playing the video through VLC. Instructions for using alternative media players are provided under [Advanced Scenarios](#advanced-scenarios).

tl;dr
```
streamlink --retry-streams 10 --player mpv -r stream.mp4 <stream_url> best
```

### Prerequisites
1. Install [Streamlink](https://streamlink.github.io/install.html)
2. Install [VLC](https://www.videolan.org/vlc/) or another video player of your choice.
3. Optionally install [youtube-dl](https://youtube-dl.org/). A simple install and usage guide is provided [here](README.md). This will be used to download the video description and thumbnail for a more complete archive.

### Instructions
The following are basic instructions for recording a livestream. The example will save the stream as `stream.mp4` in the folder `C:\Users\anon\stream_folder`. The highest quality stream will be used. 

1. Create a folder for where you want to save the stream to.
2. Open the command prompt. You copy and paste commands here and press enter to run them.
    * Open start menu, search for command prompt.
3. Move to the folder where you want to save the stream to.
    * Command: ```cd "path_to_folder_from_step_1"```
    * Example: ```cd "C:\Users\anon\stream_folder"```
4. Start recording the stream
   * Command: ```streamlink -r <name_to_save_stream_as>.mp4 <stream_url> best```
   * Example: ```streamlink -r stream.mp4 https://www.youtube.com/watch?v=RPdUErEiRbk best```

### Advanced Scenarios
These are streamlink commands for various scenarios.

#### Save the stream in 720p.
This is useful when you do not have enough bandwidth for 1080p streams.

```
streamlink -r <name_to_save_stream_as>.mp4 <stream_url> 720p
```

#### Automatically start recording when the stream goes live.
When the waiting room is available, you can use this command to have streamlink try start recording every 10 seconds. This is very useful if you know you will not be present for the start of the stream.

```
streamlink --retry-streams 10 -r <name_to_save_stream_as>.mp4 <stream_url> best
```

#### Use MPV or MPC instead of VLC for video playback.
You may have to provide the full file path to the video player executable instead of just using the short name. to use the short name instead of the full file path, you have to add the file path of the video player to your `PATH`. Refer to the `setx` command [here](README.md#windows-setup)

mpv:
```
streamlink --player mpv -r <name_to_save_stream_as>.mp4 <stream_url> best
```

mpc:
```
streamlink --player mpc-hc64 -r <name_to_save_stream_as>.mp4 <stream_url> best
```

#### Only record the stream, do not play the stream.
Changing `-r` to `-o` in the command makes it only save to a file. This is useful to combine with `--retry-streams 10` to record a stream that has been scheduled when you will not be present.

```
streamlink -o <name_to_save_stream_as>.mp4 <stream_url> best
```

```
streamlink --retry-streams 10 -o <name_to_save_stream_as>.mp4 <stream_url> best
```

## Recording Members Only Streams
Coming soon. Streamlink doesn't work since it does not support logging in to a youtube account. You will have to use youtube-dl.

See:
1. [Accessing members only content](README.md#Download-a-members-only-video)
2. [Record and watch with one download using youtube-dl](#i-really-really-want-to-use-youtube-dl-how-do-i-record-and-playback-at-the-same-time)
3. When the stream ends, press `Cntrl + C` to stop `youtube-dl`. **Only do this once** or you might corrupt your recording. It will take several minutes (less than 10 usually), to gracefully exit.
4. You can try opening the recording after you hit `Cntrl + C` once. If the file opens and plays, you can make a copy of the file and exit out of `youtube-dl` with a second `Cntrl + C`.

## Post Processsing
The previous steps should have given you a working video file but it can be improved with a few simple steps. The following steps will convert the file to a real `.mp4` file [[note](#faq)], add a fancy thumbnail, save the video description with the recording, and give the recording a nice name. 

Comparison:
![Post Processing Difference](assets/post_process_difference.jpg)

Do the following in PowerShell instead of command prompt. You can convert a command prompt into PowerShell by using the command `powershell` or open it directly from the start menu.
You will have to open a new command prompt/PowerShell while your previous command prompt is busy running Streamlink.

1. Generate a nice filename to be used later. Example generated filename: `[Botan Ch.獅白ぼたん][20200830] 5期生より (fCqDv94ZeuA)`. You will want to do this step while the stream is still live.
```
$filename = youtube-dl --write-description --skip-download -o "[%(uploader)s][%(upload_date)s] %(title)s (%(id)s)" --get-filename <stream_url>
``` 
2. Download the livestream's description and thumbnail as `stream.description` and `stream.jpg` respectively. You will want to do this step while the stream is still live, once the stream ends and the archive is deleted, you will not have access to the description of thumbnail anymore.
```
youtube-dl --write-thumbnail --write-description --skip-download -o stream <stream_url>
```
2. Store the livestream's description into variable `$description`.
```
$description = [IO.File]::ReadAllText(".\stream.description")
```
3. Convert to `.mp4`, add thumbnail and add description to the video's comment.
```
ffmpeg -i .\stream.mp4 -i .\stream.jpg -map 1 -map 0 -c copy -disposition:0 attached_pic -metadata comment=$description $($filename + ".mp4")
```

## FAQ
### What do you mean by a real .mp4
The original recording from streamlink is saved with a `.mp4` file extension but it is actually a `MPEG-TS` format file. Most video players will still be able to play the recording since they understands the format and do not rely on the file extension. 

The fake `.mp4` extension is convient for most people because their computers would be set up to use a video player to open files with the extension `.mp4`. Converting it to a real `.mp4` file will reduce the filesize without affecting quality.

### My thumbnail was downloaded as a .webp file and that file format is not supported for thumbnails.
Convert from `.webp` to `.jpg`. ffmpeg is wonderful.

```
ffmpeg -i stream.webp stream.jpg
```

### I really really want to use youtube-dl, how do I record and playback at the same time?
```
youtube-dl -o - stream_url  best | tee stream.mp4 | mpv -
```
You can get `tee` by installing [git](https://git-scm.com/downloads). This will probably not work in PowerShell.

### Why not use AtomicParsley to add the thumbnail?
AtomicParsley only records the first 255 characters of the video's comment. If you want to use it, do it before adding the comment.

### How do I... ?
Read the manuals
1. https://github.com/ytdl-org/youtube-dl/blob/master/README.md
2. https://ffmpeg.org/documentation.html
3. https://streamlink.github.io/cli.html