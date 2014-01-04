---
layout: post
title:  "Automating the opening of kvis"
date:   2013-12-30
---

[karma](http://www.atnf.csiro.au/computing/software/karma/) is a software suite provided by the [ATNF](http://www.atnf.csiro.au/). I use kvis from karma frequently for viewing my astronomy data. It works well, but opening it is the bane of every radio astronomer. Unfortunately, kvis does not remember the previous layout when it was closed, or even open into some defined state. This means it must be "set up" for at least a couple of minutes, every time you want to use it.

For years I've been threatening to address this issue. Source code, if provided, looks like rather scary FORTRAN, so I was curious to investigate some kind of intelligent mouse-controlling software. Finally, after spending most of a day flying home for holidays, I think I have a solution in the form of a Ruby script.

It turns out there's a neat piece of software called [xdotool](http://www.semicomplete.com/projects/xdotool/xdotool.xhtml) for controlling an X11 cursor. It's not intelligent in the sense that it knows where it's going, what it's clicking on etc., but it's scriptable enough for use. It seems well distributed in the Linux domain, but I'm unsure of the Mac habitat. I will test it on a Mac when I return to Uni.

[xwininfo](http://linux.die.net/man/1/xwininfo) is also required, because xdotool doesn't report window geometries correctly. The only other prerequisite is a [Ruby](http://www.ruby-lang.org) interpreter, selected for it's ease of scripting. If you don't like Ruby, you could probably easily convert to Python, if you're that way inclined.

The way it works is by moving the mouse and clicking on various buttons and menus in a specific order. Coordinates for each button and so forth are determined by screenshots. Windows are easily moved and resized in the script also.

My script is far from perfect, but hopefully it's easy enough to read and alter for personal preference. Of course, I will make improvements over time. You can find it on my GitHub <s>here</s>. **EDIT: See update below.**

Finally, here it is in action. I hope this helps people.

<img class="img-responsive" src="/images/posts/kvis.gif">

If you want to set your own various kvis options, I have saved some of the screenshots for default kvis windows as png files [here](https://www.dropbox.com/sh/mxn86dhuk34hvet/VuC_AqEQDz).

* * *

### Updated: 04/01/14
I've now moved this project into it's own GitHub repo [here](https://github.com/cjordan/kvis-setup). Please follow the Readme and discuss any issues you have over there.
