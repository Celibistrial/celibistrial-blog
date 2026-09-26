---
title: "Fastest lock screen using i3lock"
description: "A tiny bash script that blurs your desktop and locks it with i3lock in under a second."
date: "2024-01-01"
yearOnly: true
tags: ["blog"]
---

This script takes a screenshot of your desktop, blurs it, and locks the screen with the blurred image as the background. The whole thing runs in under a second.

You need `scrot`, ImageMagick (for `convert`), and `i3lock`.

```bash
#!/bin/bash
PICTURE=/tmp/i3lock.jpg

scrot --silent "$PICTURE"
convert -scale 10% -blur 0x0.5 -resize 1000% "$PICTURE" "$PICTURE"
pkill i3lock
i3lock -i "$PICTURE"
rm "$PICTURE"
```

## How it works

`scrot` saves a screenshot to `/tmp/i3lock.jpg`.

The `convert` line does the blur. Shrinking the image to 10% and scaling it back up by 1000% throws away most of the detail, which gives the pixelated look, and `-blur 0x0.5` softens the blocky edges a little. This is much faster than a real Gaussian blur on a full-size screenshot, and it's the reason the script is quick.

`pkill i3lock` kills any lock screen that's already running, so you never end up with two. Then `i3lock -i` locks the screen with the blurred image.

i3lock forks into the background once it has loaded the image, so `rm` runs straight away and deletes the screenshot from `/tmp`.

Save it somewhere like `~/.local/bin/lock`, make it executable, and bind it to a key in your i3 config.

I use JPG instead of PNG because it's faster. In my testing PNG was almost 2 seconds slower.
