---
layout: post
title: I made a Joplin -> Jekyll importer
date: 2026-01-23 03:20 +0000
author: itsrealfake
tags: []
---
While I was waiting for some code to finish executing I vibe'd up a thing. It helps me because I've got [Joplin](https://joplinapp.org) on all my devices, sync'd up with [Syncthing](https://syncthing.net/).  
<br/>You can see the tool if you want, it's [on that centralized provider](https://github.com/itsrealfake/import-joplin-to-jekyll) for now

Here's a first look

![a2f2b727513551706840dfd85ba54595.png](/assets/2026-01-23 03:20:23Z/a2f2b727513551706840dfd85ba54595.png)

I can write in Joplin  
<br/>![d27e9b3779e5f82c2eaf956f457614b8.png](/assets/2026-01-23 03:20:23Z/d27e9b3779e5f82c2eaf956f457614b8.png)

Then export to to a directory like so:  
<br/>![2e139da265ab929f0af7a5dbe8d01055.png](/assets/2026-01-23 03:20:23Z/2e139da265ab929f0af7a5dbe8d01055.png)

And then it works!

![38b8f0e283447a7f5ae73978bc93b4ef.png](/assets/2026-01-23 03:20:23Z/38b8f0e283447a7f5ae73978bc93b4ef.png)

Then i can convert it to a post with \`bundle exec jekyll publish \_drafts/i-made-a-joplin-jekyll-importer.md\`

![4384d646091acaf80ff49739adf53597.png](/assets/2026-01-23 03:20:23Z/4384d646091acaf80ff49739adf53597.png)
