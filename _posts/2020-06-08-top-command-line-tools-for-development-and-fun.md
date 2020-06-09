---
layout: post
title: Top Command Line Tools for Development and Fun
categories: [homebrew, tldr, ag, tig, tree, curl, httpie, jq, glances, wrk, youtube-dl, cmus, cli]
---

As a software developer I love working in the terminal where I do most of my work.

iTerm, zsh, vim, tmux became my favorite tools but there are many others I use pretty much on a daily basis. 

Let's take a look at some of them.

## Homebrew

This one you probably already know. [Homebrew](https://brew.sh/) is a *fantastic* package manager for MacOS and Linux with an active community of developers on Github.

You can install nearly any software using Homebrew. Every command line tool described in this blog post below can be installed as easy as running
```bash
brew install <tool name>
```

Homebrew also allows you to install Desktop applications:
```
brew cask install firefox
```

To search for particular software run:
```bash
brew search postgres
```

<!-- more -->

To install Homebrew itself from the command line run:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

## tldr

[tldr pages](https://tldr.sh/) is a collection of simplified man pages. It's great when you need to recall how to use a certain command or tool but don't want to search through a man page or documentation.

```bash
tldr tar
```

Output:
![tldr](/images/tldr.png)

## ag

[ag](https://geoff.greer.fm/ag/) aka The Silver Searcher is a tool for searching code.

I've been using [ack](https://beyondgrep.com/) for years but recently switched to ag as it works much faster (34 times in some tests). 

For example, use it to list files in the current directory that contain printf:
```bash
ag -l printf
```

Install it with Homebrew as:
```bash
brew install the_silver_searcher
```

## tig

[tig](https://jonas.github.io/tig/) is an ncurses-based text-mode interface for git.

It works mainly as a Git repository browser, but can also assist in staging changes for commit at chunk level and act as a pager for output from various Git commands.

tig is super fast and it's easy to navigate around.

![tig](/images/tig.png)

## tree

[tree](http://mama.indstate.edu/users/ice/tree/) is a great little tool that prints content of directories in a tree like structure format.
I like to use it to get a quick overview of an unfamiliar project, for example, when I clone something from Github.

For example, let's print directories only up to 3 levels of depth:
```bash
tree -d -L 3
```
Output:
![tree](/images/tree.png)

## curl / httpie

[curl](https://curl.haxx.se/) is a command-line HTTP client. It's oldie but goodie and there are some modern alternatives with colorful output and which are probably more user friendly (see the list below). 
I have them installed on my machine but always forget to use in favor to curl.

A few curl commands I use often:

See your public ip address:
```bash
curl ifconfig.co
```

Follow redirects and display response headers of a particular site:
```bash
curl -L -I http://google.com
```

Download a file:
```bash
curl -O <url>
```

Modern alternatives:
* [httpie](https://httpie.org/)
* [curlie](https://curlie.io/)

## jq

[jq](https://stedolan.github.io/jq/) is a lightweight and flexible command-line JSON processor.

jq is like sed for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that sed, awk, grep and friends let you play with text.

It plays nicely with curl.

GitHub has a JSON API, we can get the latest commit from the Ruby on Rails repository and feed it to jq:
```bash
curl https://api.github.com/repos/rails/rails/commits?per_page=1 | jq
```

Output:
![jq](/images/jq.png)

Now, let's extract the message of the commit:
```bash
curl https://api.github.com/repos/rails/rails/commits?per_page=1 | jq '.[0] | .commit.message'
```

Output:
```bash
"Merge pull request #39573 from etiennebarrie/remove-invalid-autoload\n\nRemove invalid autoloads"
```

## glances

[glances](https://nicolargo.github.io/glances/) is a system monitoring tool. You can think of it as `top` (`htop`) on steroids. 

It gives a large amount of monitoring information about your system: 
* CPU, memory and disk space usage
* Running process
* Running Docker containers
* Networks, sensors, wifi, etc.

You can install additional plugins to monitor a bunch of other things.

Here is how it looks:
![glances](/images/glances.png)

You can set up monitoring for certain processes. For examples, to monitor Ruby applications create ~/glances.ini with the following content:
```bash
[amp_ruby]
enable=true
regex=.*ruby.*
refresh=1
```

## wrk

[wrk](https://github.com/wg/wrk) is a handy HTTP benchmarking tool. It's like ApacheBench (ab) but a modern version.

It's capable of generating significant load when run on a single multi-core CPU.

For example, let's run a benchmark for 60 seconds, using 8 thread, and keeping 400 HTTP connections open against a simple HTTP server implemented in Go:
```bash
wrk -t8 -c400 -d60s http://localhost:8080/hello
```

Output:
```bash
Running 1m test @ http://localhost:8080/hello
  8 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.56ms    2.26ms  95.31ms   92.00%
    Req/Sec     8.77k     2.24k   15.75k    64.66%
  4190457 requests in 1.00m, 535.51MB read
  Socket errors: connect 157, read 100, write 0, timeout 0
Requests/sec:  69757.81
Transfer/sec:      8.91MB
```

## Youtube-dl

[youtube-dl](https://ytdl-org.github.io/youtube-dl/index.html) allows you to download videos from YouTube and [many](https://ytdl-org.github.io/youtube-dl/supportedsites.html) other sites.

It has multiple options, supports playlists, can extract audio, download subtitles.

A typical scenario for me is to download videos when we plan a long trip where we might have no Internet connection and to put them on iPad via iTunes. This works great as I don't really watch movies on a plane but Youtube has plenty of screencasts and educational materials.

Another case is when I go for a run and want to listen to a Youtube video. It turns out the visual part doesn't play a huge role and you can simply listen to many videos.

youtube-dl can work with Youtube urls but I find it easier to supply it with video ids. This is the part that goes after ?v=. For example: _grnQ46lNDAc_ in the following video:
https://www.youtube.com/watch?v=grnQ46lNDAc

Some commands:

Download a Youtube video in the mp4 format:
```bash
youtube-dl grnQ46lNDAc 
```

Download video, extract audio and save in the mp3 format:
```bash
youtube-dl -extract-audio \
    --audio-format mp3 UtF6Jej8yb4
```

Download video, extract audio and save in the mp3 format with meta data:
```bash
youtube-dl --extract-audio \
    --audio-format mp3 \
    --metadata-from-title "%(artist)s - %(title)s" \
    --add-metadata sR5uT4k-QRc
```

## cmus

[cmus](https://cmus.github.io/) is small, fast and powerful console music player.

I could never get used to iTunes on Mac or any other Desktop music player but cmus works just great for me -- it's light, has a clean interface, Vim key bindings, has many options and supports multiple music formats.

![cmus](/images/cmus.png)

To start using it go to the folder with your music files and type `cmus`
Then `:add .` to add music form the current directory.

As mentioned above you can use Vim key bindings to navigate around. 

A few useful keys:
```
1, 2, 3, 4, 5, 6, 7 - change views
x - play track
c - pause track
v - stop track
b - next track
z - previous track
```

To change a color scheme you can use `:colorscheme` (solarized-light, anyone?)

Well, that's all for today.

There are a bunch of other great command line tools. What are yours? I would love to hear from you!

