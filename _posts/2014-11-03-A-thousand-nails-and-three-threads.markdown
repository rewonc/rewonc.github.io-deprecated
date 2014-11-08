---
layout: post
title:  "A thousand nails and three threads: part one"
date:   2014-11-03 07:05:06
categories: update
---

Some time ago I came across the "Constellation" pieces by artist Kumi Yamashita and couldn't forget them. Check out her [online gallery](http://www.kumiyamashita.com/constellation/) to see what I mean--they're incredible. Taking a white board, thousands of small nails, and a single sewing thread, she weaves portraits of people around her by weaving the thread between the nails. The components are simple but the result is nuanced, fluid, organic, and lifelike. 

It just so happens that a few weeks ago I moved into a huge loft in Emeryville. Being a former airplane engine manufacturing plant, the loft has towering walls that cry out, _put art on me!_. Now ordinarily in furnishing a loft's walls one either needs to have the money to purchase art or needs to have the intution, perseverance, and raw artistic ability that characterize someone like Kumi Yamashita to create the art. 

I turn out to have none of those things, but I know Javascript, which has followed [Atwood's Law](http://blog.codinghorror.com/the-principle-of-least-power/) for the most part and now can be used for most tasks a programmer would like to accomplish.  So I thought, why not try and generate some fine art using Javascript?

So that was my programming assignment for the last week. Create a script that would analyze an image and output instructions for recreating it in real life with nails and thread a la Yamashita. And also, why not do it in color, too?

This seemed like a project that would turn out to be much harder than I originally anticipated, but I decided to do it anyway. This post details the algorithm I came up with. The physical construction of the piece will be detailed in a following post. 

##Tools

HTML5 Canvas actually has a couple very useful methods for us:   [getImageData]( https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#getImageData ) and [putImageData](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#putImageData )

These are getters and setters for the ImageData property of the HTML canvas. They return an array that has the red, green, blue, and alpha (opacity) value for each pixel in the image. This allows us to create a model of the image as an array and run any operations we might want on it (for example, drawing a thread), then push it back to the canvas to render. Sounds like just what we need to complement our plain ol Javascript toolkit.

##First approach

I figured I would first go with the simplest, most naive approach I could think of then optimize later. Here's the non-functional pseudocode of what I came up with:

{% highlight javascript %}

var pixels = getRGB("path/to/img.jpg");
var grid = generateSimpleRectangularNailGrid({width: 30, height: 54});
var canvas = document.getElementById("canvas");

//Set a maximum limit corresponding to what a human could draw in a couple weeks
var MAX_LINES_DRAWN_PER_THREAD = 1500; 
//Set the "stroke intensity"--on a scale of 0 to 255, how intensely red, green, or blue will each stroke be?
var STROKE_INTENSITY = 64;

var drawColor = function(color, countdown, next){
  if (countdown <= 0 ) return;
  var origin = next || Grid.getRandom(grid);
  var line = RandomlyFindAValidNextLine(grid, pixels, STROKE_INTENSITY);
  if (typeof line === "number")  {
    addToGrid(line, grid);
    renderNewLineOnCanvas(line, canvas, STROKE_INTENSITY);
    decreasePixelsInPlace(line, pixels, STROKE_INTENSITY);
    return drawColor(color, count - 1, line.end);    
  } else {
    return drawColor(color, count);
  }
}

drawColor("red", MAX_LINES_DRAWN_PER_THREAD);
drawColor("green", MAX_LINES_DRAWN_PER_THREAD);
drawColor("blue", MAX_LINES_DRAWN_PER_THREAD);

{% endhighlight %}

[The repo with the functional source code is available here](https://github.com/rewonc/nailsandthread) 

If you just skipped over the code, the basic gist is that it chooses a point on the grid at random for the origin, then randomly guesses another point until it finds one that is valid to draw. Once it finds a valid point, it draws a line to that point, then repeats. If it can't find a valid point, it chooses another one at random.

What is a valid line? For a line to be valid, it needs to have two endpoints (nodes on the grid) and all the pixels in between those endpoints must have red, green or blue values higher than the STROKE_INTENSITY value.  Make sense?  So, if you have a stroke intensity of 64 and are drawing "red", the algorithm will only draw in places where all the pixels underneath have at least a value of 64 for red. 

Here's what this algorithm draws when applied to an image of... can you guess?

###First try render
![Little doggy woggy]({{ site.url }}/img/doggy-1403.png)

Did you guess... some kind of animal, wearing a bow? A puppy perhaps? Good job! Here's the source image showing the subtractions the algorithm has made from this ridiculously cute dog:

![Little doggy woggy source]({{site.url}}/img/doggysrc.png)

Not bad for a totally random algorithm. I let it run for about 1-2 minutes before stopping it. (In the real code, I had it take a 0.5 second break after each set of lines so browser wouldnt kill the script, so its not 1-2 mins of straight computation). Here's some things I learned from implementing this simple algorithm:

* It didn't actually complete. It only drew 1403 lines before I stopped it. The random search method drew lots of lines initially but slowed down around the ~1000 mark, suggesting that I could likely improve the sampling function. 

* Even though there's only 1403 lines, which represents a tiny fraction of the information in the image, the dog is still recognizable for the most part. This means we probably don't need to draw _all_ the image information to get a good enough representation of it.

* It doesn't do too well around the edges. The feet, for example, look polygonal and and the fluffy edges aren't rendered so well. It looks like the square grid of nails doesn't map 1:1 to the contours of the dog. 

Ok, that's a good list of stuff to begin improving.

##Second try

###Low-hanging fruit

So as we didn't actually reach our target of successfully drawn lines, an easy way to improve the quality would be to add more nodes and lighten the stroke on the lines to get more lines drawn.

{% highlight javascript %}

//double the dimensions of our grid (4x more nodes)
var grid = generateSimpleRectangularNailGrid({width: 60, height: 108});
//Halve the strength of the stroke
var STROKE_INTENSITY = 32;

{% endhighlight %}


Here's the results from that (left: before, right: after).  This time, it easily drew ~4500 lines:

![Little doggy woggy]({{ site.url }}/img/doggy-1403.png)
![Higher res doggy woggy]({{ site.url }}/img/doggy-double-rendered.png)

Ok, getting better, although now it looks like the result of a kid scribbing with crayons. Oh well, let's move on, keeping in mind that we now can adjust the number of nodes and stroke intensity to adjust the quantity of lines drawn. 

### Manipulating the angle of lines drawn
Compared with [Yamashita's originals](http://www.kumiyamashita.com/constellation/), the average line length in our current portrait is long.  Yamashita generally seems to pass the thread between adjacent nodes rather than across the grid. When we sample for points across the entire grid, we end up getting a lot of long lines that go across a long part of the grid and result in a "scribbled" look. Let's see if we can't adjust that:

{% highlight javascript %}
//radius refers to the maximum of nodes you will sample. 
//For example, putting 1 for a node in the middle of a grid will return the 9 nodes surrounding it.
var radius = 1;
var list = findNodesAdjacentTo(origin, grid, radius, color);
var next = _.find(list, validatePoint);
if (next !== undefined) { /* draw & recur */ }
else                    { /* Start over */}
{% endhighlight %}

Here's what we get from doing that. 

Radius 1 & 2:

![Little doggy woggy]({{ site.url }}/img/angle1.png)
![Little doggy woggy]({{ site.url }}/img/angle2.png)

Radius 3 & 5:

![Little doggy woggy]({{ site.url }}/img/angle3.png)
![Little doggy woggy]({{ site.url }}/img/angle5.png)

Maximum radius (sampling only the points that are farthest away) & random radius (weighted to a power curve so lower values appear much more frequently)

![Little doggy woggy]({{ site.url }}/img/anglemax.png)

Kind of neat!  As it turns out, in our square grid of nails changing the sampling radius changes the 

### Increasing line connectivity

{% highlight javascript %}
var x = 0;
{% endhighlight %}


And here's the result: 


###In the next installment:

- Turning this into an accurate representation of a physical canvas

- Semi-random, organic node placement

- Averaging over pixel groups

- Constructing it in real life 







