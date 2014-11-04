---
layout: post
title:  "A thousand nails and three threads: part one"
date:   2014-11-03 07:05:06
categories: update
---

Some time ago I came across the "Constellation" pieces by artist Kumi Yamashita. In case you haven't seen them, you can look at them at her [online gallery](http://www.kumiyamashita.com/constellation/). The portraits are marvelous to look at and incredibly intricate. It's wonderful how something so fluid and organic can emerge from simple mechanical components.

A couple weeks ago I was thinking about these portraits, and it just so happened that around the same time I had 1) been doing a fair amount of self study on graph transformation algorithms for a separate project and 2) had recently moved into a new loft apartment in Emeryville, CA with very bare, very large walls. As a way of challenging my understanding of the material I was learning and also furnishing my apartment, I asked myself--why not recreate something similar on my walls? I could just choose a photo to erect, write a small script to tell me the order I should weave, then get some nails and some thread and put it up. Sounds easy enough right?

As I was enlisting the help of a computer to do this all, moreover, I figured I'd add an extra challenge: make the portrait in color.  As images are often represented as maps of pixels with distinct values for their red, green, and blue values, I figured that with one red, one green, and one blue thread I ought to be able to render a full color portrait.  

I might as well spoil it for you--it wasn't as easy as I thought. It also took a lot longer than I thought. But I'm quite happy with the finished product:



I'll go into some detail about how this worked for anyone interested in doing a similar project.

##The code

####Specifications

If you are someone like the original artist, you might be able to create a piece like this from intuition, perseverance, or raw artistic ability. I suffer from a lack of all three, and so I wanted to write a piece of code that would specify exactly what I had to do and when so I had to make no complicated judgments while making the portrait.  

More specifically, I wanted to know:

- How many nails should I use? Where should I put them?
- How much string do I need to buy? What thickness?
- Where should each thread begin, and what path does it take?
- Does the order of placing the strings matter?
- How long is it going to take to put this together?

Additionally, I wanted a way to quickly visualize the result before I started. In other words, the code needed to include a rendering component that would approximate what the finished result would look like. 

####Tools
For the visualization piece, it seemed to be a no-brainer to use [D3.js](http://d3js.org/). It's familiar, sexy, popular--what else would you look for in a library? 

For the processing piece, it seemed like there were more options. There are a couple libraries -- notably ImageMagick -- that offer options for image manipulation on the server side in Ruby, Python, Clojure, or whatever. For this project, however, all I needed to do was convert the image to an array of RGB elements and run some straightforward operations. It turns out there's an [HTML5 Canvas method getImageData](https://html.spec.whatwg.org/multipage/scripting.html#dom-context-2d-getimagedata) that does the RGB conversion for you.   So, if we can use HTML5 Canvas to get the image data, and want to render it with D3.js, it looks like we should just do everything else on the browser too. Go Go Javascript!

####Naive approach

I figured I would go with a simple, naive approach first then optimize later. Here's the pseudocode of what I came up with:

<pre><code>
/* load image and convert to array */
var pixels = loadImageData('path/to/source.jpg');

/* settings */
//How many lines do I want to draw? When I'm drawing the actual portrait, let's assume I can do 1 line per 5 seconds 
and want to spend 30 minutes per day for 2 weeks. That means ~5000 lines maximum. 
var MAX_LINES_DRAWN = 5000

</code></pre>
(The repo with all the real source code available here)


###The demo

###The supplies

###The timeline

###The finished product

