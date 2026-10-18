# Browser-free YouTube

2026-10-15 <i>last updated 2026-10-18</i>

In curtailing the addiction that advertising companies such as Google and Meta push upon us, browser-free YouTube is a gem.
Its implementing seems fragile and rigorous due to the nature of YouTube's API, so I describe my current working method.
It uses a UNIX command-line environment.

## Prerequisites

* [Newsboat](https://newsboat.org/)
* [Docker](https://www.docker.com/)
* [mpv](https://mpv.io/)
* [yt-dlp](https://github.com/yt-dlp/yt-dlp)

Note on yt-dlp: I am using Debian stable. Stability is undercut by the YouTube API's breaking changes, requiring breaking changes in some of the requisite software.
While yt-dlp exists in Debian's package repository, and will even auto-install with the mpv package, the provided version is too out-of-date.
I had to manually install the latest version of yt-dlp. Thankfully there is [a single python executable released on GitHub](https://github.com/yt-dlp/yt-dlp/releases/latest).
I put `yt-dlp` into `~/.local/bin`. If you do this, make sure `~/.local/bin` is before `/usr/bin` in your `PATH` environment variable.

In sum, I executed:

```
# apt install newsboat docker-cli docker.io mpv
$ wget -O ~/.local/bin/yt-dlp https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp
$ chmod +x ~/.local/bin/yt-dlp
```

Note on filepaths: I am assuing the environment variable `XDG_CONFIG_HOME` is set to `~/.config`. I recommend this practice anyways.

## Procedure

### 1. Proof-of-origin token provision

Assuming the prerequisites, we must install [bgutil-ytdlp-pot-provider](https://github.com/Brainicism/bgutil-ytdlp-pot-provider), a yt-dlp plugin for generating proof-of-origin tokens for streaming.

On Debian, pip can only install python packages in a virtual environment, so I manually install:

```
$ mkdir -p ~/.config/yt-dlp/plugins
$ wget -O ~/.config/yt-dlp/plugins/bgutil-ytdlp-pot-provider.zip https://github.com/Brainicism/bgutil-ytdlp-pot-provider/releases/latest/download/bgutil-ytdlp-pot-provider.zip
```

The plugin operates with an HTTP server, which we run in a Docker image. Download the image and run:

```
# docker pull brainicism/bgutil-ytdlp-pot-provider
# docker run --name bgutil-provider -d --init -p 127.0.0.1:4416:4416 brainicism/bgutil-ytdlp-pot-provider
```

### 2. Configuring Newsboat

We create a macro in newsboat to open links with mpv, which automatically uses yt-dlp, which in turn will now automatically connect to our Docker-ized HTTP server for PO token generation.

In `~/.config/newsboat/config`, add the line:

```
macro m set browser "mpv %u" ; open-in-browser ; set browser "lynx -accept_all_cookies -vikeys %u"
```

This line resets the browser to [Lynx](https://lynx.browser.org/) a terminal-based web browser. You can replace the string after `set browser` to something like `"firefox %u"` or `"$BROWSER %u"`. If you are making the config file for the first time, add the line `browser ...` with the browser string you want before the line above (no `set`). None of this matters if you will only use Newsboat for YouTube.

[A complete config file of my own design](https://raw.githubusercontent.com/tteeoo/usr/refs/heads/main/home/.config/newsboat/config), which contains the above line but also sets vi-like controls and color changes can be found on GitHub.

Add your YouTube subscriptions to `~/.config/newsboat/urls` line-by-line in the following format:

```
https://youtube.com/feeds/videos.xml?channel_id=ChannelID "~Name"
```

The channel ID can be found in the URL of the channel's page, or pressing "...more" on the channel bio, and navigating to "Share channel" > "Copy channel ID". It looks something like `UCY2mhw-XNZSxrUynsI5K8Zw`.

Now, update newsboat with `newsboat -x reload` and then with no options to open it and see your feeds. Enter one, enter onto a video, and press `,` then `m` to run our macro. mpv should open, streaming the video.

You may create a short script to cleanly start (if necessary) the Docker container, update and open newsboat, then stop the Docker container.

```
#!/usr/bin/bash
sudo docker ps --filter "ancestor=brainicism/bgutil-ytdlp-pot-provider" --format '{{.ID}}' | grep -q . || sudo docker run -d brainicism/bgutil-ytdlp-pot-provider
newsboat -x reload
newsboat
sudo docker stop $(sudo docker ps -q --filter "ancestor=brainicism/bgutil-ytdlp-pot-provider")
```

I additionally limit my streaming format for better performance by adding the following line to `~/.config/mpv/mpv.conf`:

```
ytdl-format=bestvideo*[height<=1080][fps<=30]+bestaudio
```
