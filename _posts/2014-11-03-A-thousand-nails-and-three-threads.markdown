---
layout: post
title:  "A thousand nails and three threads: part one"
date:   2014-11-03 07:05:06
categories: update
---

Some time ago I came across the "Constellation" pieces by artist Kumi Yamashita. They're awesome--check out her [online gallery](http://www.kumiyamashita.com/constellation/). Taking a white board, thousands of small nails, and a single sewing thread, she weaves marvelously intricate portraits of people around her. I find it amazing how she can turn a few simple componenet materials into something that seems so lifelike, organic, and real.

It just so happens that a few weeks ago I moved into a big loft in Emeryville, CA. The ceilings are about 20 feet tall and cry out "Put art on me!"  If you have the kind of intuition, perseverance, and raw artistic ability that Yamashita has, these walls are probably a great opportunity to unleash your inner Picasso. I suffer from a lack of all of those things, however, and as I can't afford to buy anyone else's art either, I thought the walls would remain barren for my time here. So goes my dreams for calling it an _artist's loft_. 

But after ruminating on it for some time, I realized, _I do know Javascript_.  And Javascript makes dreams come true. Perhaps, with a little bit of Javascripting, I could write something that would substitute for artistic talent, perseverance, and intuition and allow me to furnish my walls like a real artist would. 

So that was my programming assignment for the last week. Create a script that would analyze an image and output instructions for recreating it in real life with nails and thread a la Yamashita. And also, why not do it in color, too?

This seemed like a project that would turn out to be much harder than I originally anticipated, but I decided to do it anyway. This post details the algorithm I came up with. The physical construction of the piece will be detailed in following post. 

##Tools

HTML5 Canvas actually has all the image processing methods we need:   [getImageData]( https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#getImageData ) and [putImageData](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#putImageData )

These are getters and setters for the ImageData property of the HTML canvas. They return an array that has the red, green, blue, and alpha (opacity) value for each pixel in the image. So you can pull the data, run some operations on it, then push back to the canvas to render.

That really should be all we need. So we can do this whole thing in plain ol' Javascript and HTML5. 

##First approach

I figured I would go with a simple, naive approach first then optimize later. Here's the non-functional pseudocode of what I came up with:

{% highlight javascript %}

var pixels = getRGB("path/to/img.jpg");
var grid = generateSimpleRectangularNailGrid({width: 30, height: 54});
var canvas = document.getElementById("canvas");

//Set a maximum limit corresponding to what a human could draw in a couple weeks
var MAX_LINES_DRAWN_PER_THREAD = 1500; 

var drawColor = function(color, countdown, next){
  if (countdown <= 0 ) return;
  var origin = next || Grid.getRandom(grid);
  var line = RandomlyFindAValidNextPoint(grid, pixels);
  if (typeof line === "number")  {
    addToGrid(line, grid);
    renderNewLineOnCanvas(line, canvas);
    decreasePixelsInPlace(line, pixels);
    return drawColor(color, count - 1, line.end);    
  } else {
    return drawColor(color, count);
  }
}

drawColor("red", MAX_LINES_DRAWN_PER_THREAD);
drawColor("green", MAX_LINES_DRAWN_PER_THREAD);
drawColor("blue", MAX_LINES_DRAWN_PER_THREAD);

{% endhighlight %}

[The repo with the fleshed-out, functional source code is available here](https://github.com/rewonc/nailsandthread) 

If you just skipped over the code, the basic gist is that it chooses a point on the grid at random for the origin, then randomly guesses another point until it finds one that is valid to draw. Once it finds a valid point, it sets that point as the origin and looks for another one to draw.  If it can't find a valid point, it chooses another one at random to be the origin.

Here's what this algorithm draws when applied to an image of... can you guess?

####First try render
![Little doggy woggy]({{ site.url }}/img/doggy-1403.png)



###The demo

###The supplies

###The timeline

###The finished product

