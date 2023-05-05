---
layout: post
title:  "Writing Chapters to a Video using ffmpeg and Python"
date:   2021-10-21 16:00:00 +0530
categories: productivity
tags:
- ffmpeg
- python
- chapters
---

Adding *Chapters* to a video file is a great way to navigate through long videos (if there is a comprehensive chapter-structure in the video, that is). One needs a *supported container* and a *supported player*. Most containers support chapters - we're going to use `mkv` since I intend to add chapters to my lecture recordings and I record them in the `mkv` format. **VLC** has support for video chapters.

### Chapters with ffmpeg

We will use **ffmpeg** to add chapters to our video. The idea is to add chapters to the video metadata, following a certain format. From the [relevant documentation][1]:
> FFmpeg is able to dump metadata from media files into a simple UTF-8-encoded INI-like text file and then load it back using the metadata muxer/demuxer.

To see the metadata associated with a video file *input.mkv* we write:
```shell
ffmpeg -i input.mkv -f ffmetadata metadatafile.txt
```
The metadata is stored in *metadatafile.txt*. For a video with no chapters, it may be as uncomplicated as:
```shell
> cat metadatafile.txt

;FFMETADATA1
encoder=Lavf58.35.101
```
To this file, we need to append chapter data, which has the format
```
[CHAPTER]
TIMEBASE=1/1000
START=0
END=60000
title=beginning chapter
```
where
  - `TIMEBASE` indicates how many parts a *second* is divided into. *1/1000* means *1000 tsu = 1 sec* (*tsu* is not really a unit XD It stands for *timestamp-unit*, something I'll use throughout the post.)
  - `START` is the timestamp for start of the chapter.
  - `END` is the timestamp for end of the the chapter. Typically this is just *1 tsu* behind the start of the next chapter, unless this is the last chapter.
  - `title` is the name of the chapter of our choice.[^titlenote]

For each chapter we want to put in, we need to repeat this block.

After all that work, to write the metadata back to the video file, we can write:
```shell
ffmpeg -i input.mkv -i metadatafile.txt -map_metadata 1 -codec copy output.mkv
```

However, it's inconvenient to both write all this stuff repeatedly for each chapter and video *and* convert from our standard sexagesimal representation of video time *hh:mm:ss.uuu* (*uuu* stands for microseconds) to *tsu* units. A good way to scale this issue is to use a scripting language to do all of it for us. For instance, we want to provide the following data to the script:
```
5:30 Hello World
9:11 Bye World
```
and expect it to write out the metadata file accordingly.

### The Algorithm:
  
  - **Provide chapters in a text file**: Timestamp format is sexagesimal, but flexible. In *hh:mm:ss.uuu*, the *seconds* value must always be given. *hh, mm* and *uuu* are optional and will be interpreted accordingly. This allows for convenient timestamping of any (normal) video duration. 
  - **Get metadata of the video file**
  - **Get the timestamp of the end of the video file:** This can be done using **ffprobe** ([FFprobeTips][2]):
  ```shell
  ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 input.mkv
  ```
  `-v error` avoids irrelevant output from **ffprobe**. `-show_entries format=duration` selects only the *duration* of the *container*. `-of default=noprint_wrappers=1:nokey=1` is formatting that ensures that only the *value* of the *duration* is printed, without any extra text. Output is in *seconds*, typically a `float` value.
  - **Read and process chapters file:** Split lines into (starting) *timestamps* and *chapter name*. Convert sexagesimal format to *tsu* units assuming `TIMEBASE=1/1000`.
  - **Write out metadata:** Append the form of the block described and repeat for each chapter. Write to video file.






[^titlenote]: Certain special characters (`=`, `;`, `#`, `\` and a `newline`) need to preceded with a backslash `\`. Also, any spaces after `title=` will be part of the chapter name. Hence there's no need to include a space right after `=`.







[1]: <https://ffmpeg.org/ffmpeg-formats.html#Metadata-1>
[2]: <http://trac.ffmpeg.org/wiki/FFprobeTips>