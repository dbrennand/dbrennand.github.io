---
title: "Use youtube-dl to download a streamed video"
date: 2021-12-19T22:21:42Z
draft: false
tags: ["youtube-dl", "download", "streamed", "video"]
showToc: true
---

Hi there! :wave:

In this blog post, I will show you how to download a streamed video from a website using [youtube-dl](https://github.com/ytdl-org/youtube-dl).

# Background

Recently I needed to download a streamed video from a website for archival purposes. Full disclaimer, the service had already been paid for and I wanted to keep the video which was set to expire after a specific date.

# Investigation :mag:

At first there was no obvious way to download the video. Investigating the HTTP traffic in the browser, I could see that the video was being streamed with each part being progressively downloaded.

Each part of the video was split into Transport Stream (`.ts`) files, each lasting around 25 seconds. After some quick googling I came across this [stack overflow](https://stackoverflow.com/questions/22188332/download-ts-files-from-video-stream) post. The answers said to find the playlist file (`.m3u8`) associated with the Transport Stream.

# Finding the Transport Stream Playlist File URL :link:

To find the Transport Stream playlist file URL, follow the steps below:

1. Open your browser of choice.

2. Press `CTRL + SHIFT + I` to bring up developer tools and select **Network**.

3. Go to the web page containing the streamed video and use the filter to search the HTTP requests for `playlist.m3u8`.

In my example, the Transport Stream playlist file URL was similar to: `https://x.cloudfront.net/x_vods/_definst_/xvods/x-vod/vb/11111_2021-11-22_12355F4.mp4/playlist.m3u8`

# Downloading a Streamed Video using youtube-dl ⬇️

To download the streamed video I used [mikenye's youtube-dl container image](https://github.com/mikenye/docker-youtube-dl).

## Verifying the Transport Stream Playlist File URL :link:

To verify that you have the correct playlist file URL, run youtube-dl with the `-F` option to list all available formats:

```bash
docker run \
    --rm -i \
    -e PGID=$(id -g) \
    -e PUID=$(id -u) \
    -v "$(pwd)":/workdir:rw \
    mikenye/youtube-dl -F \
    https://x.cloudfront.net/x_vods/_definst_/xvods/x-vod/vb/11111_2021-11-22_12355F4.mp4/playlist.m3u8
```

This should output something similar to:

```bash
[generic] playlist: Requesting header
[generic] playlist: Downloading m3u8 information
[info] Available formats for playlist:
format code  extension  resolution note
556          mp4        640x360     556k , avc1.42c01e, mp4a.40.2
```

Awesome! 😎 Now you know you have the correct URL!

## Downloading the Streamed Video ⬇️

To download the streamed video, run the following command providing the format code from the output above:

```bash
docker run \
    --rm -i \
    -e PGID=$(id -g) \
    -e PUID=$(id -u) \
    -v "$(pwd)":/workdir:rw \
    mikenye/youtube-dl --format 556 \
    https://x.cloudfront.net/x_vods/_definst_/xvods/x-vod/vb/11111_2021-11-22_12355F4.mp4/playlist.m3u8
```

You should see youtube-dl identify each `.ts` file in the playlist, read its contents and combine them into a single `.mp4` file.

```bash
[generic] playlist: Requesting header
[generic] playlist: Downloading m3u8 information
[download] Destination: playlist-playlist.mp4
...
[https @ 0x561d074eadc0] Opening 'https://x.cloudfront.net/x_vods/_definst_/xvods/x-vod/vb/11111_2021-11-22_12355F4.mp4/media_w123456789_2.ts' for reading
...
[https @ 0x561d074eadc0] Opening 'https://x.cloudfront.net/x_vods/_definst_/xvods/x-vod/vb/11111_2021-11-22_12355F4.mp4/media_w123456789_308.ts' for reading
...
frame=77069 fps=2670 q=-1.0 Lsize=84894kB time=00:51:22.79 bitrate=225.6kbits/s speed=107x
video:35021kB audio:48855kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 1.213379%
[ffmpeg] Downloaded 86931123 bytes
[download] 100% of 82.90MiB in 00:30
```

Absolute magic! :sparkles:

I hope someone finds this blog post useful and it saves them a bit of time 🙂
