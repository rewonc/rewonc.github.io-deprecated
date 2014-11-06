---
layout: post
title:  "A thousand nails and three threads: part one"
date:   2014-11-03 07:05:06
categories: update
---

Some time ago I came across the "Constellation" pieces by artist Kumi Yamashita. In case you haven't seen them, you can see them at her [online gallery](http://www.kumiyamashita.com/constellation/). She crafts the pieces with a white board, thousands of small nails, and one single unbroken sewing thread.  The portraits are marvelously intricate. It's wonderful how something so fluid and organic can emerge from three simple components.

A couple weeks ago I was thinking about these portraits, and it just so happened that around the same time I had moved into a new loft apartment in Emeryville, CA with very bare, very large walls. As I was learning about some graph algorithms for a separate project, the idea struck me that I could write an algorithm that could calculate a path of lines and output instructions that, if followed, might allow me to craft something that approximated Yamashita's genius on my own walls. In other words, I could practice some computer science and furnish my apartment at the same time.

As I was enlisting the help of a computer, moreover, I figured I would spice up the challenge a bit. Why not try and recreate a portrait in color? So the challenge I put forth to myself was: given a few sewing threads of base colors, a white board, and a bunch of nails, construct a wall-sized color portrait a la Kumi Yamashita.

This seemed like a project that would turn out to be much harder than I originally anticipated, but I decided to do it anyway. This post details some of the methods and the algorithm I came up with. The actual construction of the piece will be in a following post. 

##Specifications

####Output

If you are someone like the original artist, you might be able to create a piece like this from intuition, perseverance, or raw artistic ability. I suffer from a lack of all three, and so I wanted to write a program that would specify exactly what I had to do so I had to make no artistic decisions while creating the piece.

More specifically, I wanted to know:

- How many nails should I use? Where should I put them?
- How much string do I need to buy? What thickness?
- Where should each thread begin, and what path does it take?
- What order should I layer the strings?
- How long is it going to take to put this together?

Additionally, I wanted a way to quickly visualize the result before I started. In other words, the code needed to include a rendering component that would approximate what the finished result would look like, so I could quickly scrap it or go with something else before putting thread to board.

####Tools
Javascript & HTML5... because why not. 
HTML5 Canvas actually has some very relevant image processing methods:   [getImageData](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#getImageData()) / [putImageData](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#putImageData())

##First approach

I figured I would go with a simple, naive approach first then optimize later. Here's the pseudocode of what I came up with:

<pre><code>
/* load image and convert to array */
var pixels = loadImageData('path/to/source.jpg');

/* settings */
//How many lines do I want to draw? When I'm drawing the actual portrait, let's assume I can do 1 line per 5 seconds 
and want to spend 30 minutes per day for 2 weeks. That means ~5000 lines maximum. 
var MAX_LINES_DRAWN = 5000

</code></pre>

(The repo with the [source code for all examples is available here](https://github.com/rewonc/nailsandthread) )


###The demo

###The supplies

###The timeline

###The finished product

