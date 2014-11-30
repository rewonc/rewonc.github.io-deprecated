---
layout: post
title:  "A thousand nails and three threads: part one"
date:   2014-11-03 07:05:06
categories: update
---

Some time ago I came across the "Constellation" pieces by artist Kumi Yamashita. Check out her [online gallery](http://www.kumiyamashita.com/constellation/) if you haven't seen them--they're incredible. She takes a white board, thousands of small nails, and a single sewing thread and weaves incredibly lifelike portraits. The components are simple but the result is nuanced and organic.

I wanted to recreate something in Yamashita's style in my own apartment. I lack both artistic talent and money so normally this would not be possible, but something about the Yamashita pieces suggested to me that I might try programming an algorithm that would spit out instructions to create a piece in real life. 

I of course mean in no way to suggest that a mere algorithm could replicate Yamashita's artistic talent and perseverance in making these portraits.  Rather, I mean to say that Javascript has been adapted to do [most things a programmer would like to accomplish](http://blog.codinghorror.com/the-principle-of-least-power/) reasonably well, so why not fine art, too?

That was my assignment for last week. Create a script to analyze an image and output instructions for recreating it in real life with nails and thread a la Yamashita. And also, why not do it in color while we're at it?

##Tools

With HTML5 Canvas's  [getImageData]( https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#getImageData ) and [putImageData](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#putImageData ), we have all the tools we need with plain old Javascript. 

GetImageData returns an array that has the red, green, blue, and alpha (opacity) value for each pixel in the image. All we need to do is manipulate that with our Javascript program then push it back to the canvas with putImageData to visualize our result. 

##First approach

I figured I would first go with the simplest, most naive approach I could think of then optimize later. Here's the pseudocode of what I came up with: 

{% highlight javascript %}

var pixels = Helpers.getRGB("path/to/img.jpg");
// Generate a graph to represent a simple square grid of nails.
// Nails become nodes. Lines become edges. 
var graph = new Graph(pixels, {width: 30, height: 54});
var canvas = document.getElementById("canvas");
 
// Define some "threads"
var LINE_INTENSITY = 64;
var threads = [
  {name: "red", r: LINE_INTENSITY, g: 0, b: 0},
  {name: "green", r: 0, g: LINE_INTENSITY, b: 0},
  {name: "blue", r: 0, g: 0, b: LINE_INTENSITY},
];

var drawNextLine = function (graph, thread, previous, count) {
  if (count < 0) return;
  previous = previous || graph.getRandomNode();
  result = graph.getNextValidNodeRandomly(previous, thread);
  if (result.node !== undefined) {
    graph.addEdge(origin, result.node, thread);
    graph.decrement(result.line, thread);
    return drawNextLine(graph, thread, result.node, count - 1)
  }
  return drawNextLine(graph, thread, count - 1);
};

drawNextLine(graph, thread[0]);
drawNextLine(graph, thread[1]);
drawNextLine(graph, thread[2]);

{% endhighlight %}

[The repo with helper functions and working source code is available here](https://github.com/rewonc/nailsandthread) 

If you just skipped over the code, it basically chooses a point on the graph at random for the origin, then randomly guesses another point until it finds one that is valid to draw. Once it finds a valid point, it draws a line to that point, then repeats. If it can't find a valid point, it chooses another one at random.

What is a valid line? For a line to be valid, it needs to have two endpoints (nodes on the graph) and all the pixels in between those endpoints must have red, green or blue values higher than the LINE_INTENSITY value. So, if you have a stroke intensity of 64 and are drawing "red", the algorithm will only draw in places where all the pixels underneath have at least a value of 64 for red. 

Here's what this algorithm draws when applied to an image of... can you guess?

###First try render
![Little doggy woggy]({{ site.url }}/img/doggy-1403.png)

Did you guess... some kind of animal, wearing a bow? A puppy perhaps? Good job! Here's the source image showing the subtractions the algorithm has made from this ridiculously cute dog:

![Little doggy woggy source]({{site.url}}/img/doggysrc.png)

Not bad for a totally random algorithm. The algorithm only drew 1400 lines and contains only a fraction of the information in the image, but you can still tell it's a dog. But we can do better on some points:

* This is pretty slow -- it took around a minute to draw the above.

* It looks pretty polygonal, and it doesn't seem to do very well with edges. 

* A lot of the lines aren't connected, which would make real life rendering difficult.

Ok, that's a good list of stuff to begin improving.

##Second try

###Low-hanging fruit

So as we didn't actually reach our target of successfully drawn lines, an easy way to improve the quality would be to add more nodes and lighten the stroke on the lines to get more lines drawn.

{% highlight javascript %}

//double the dimensions of our grid (4x more nodes)
var graph = new Graph(pixels, {width: 30 * 2, height: 54 * 2});
//Halve the line intensity
var LINE_INTENSITY = 32;

{% endhighlight %}


Here's the results from that (**left**: before, **right**: after).  This time, it easily drew ~4500 lines:

![Little doggy woggy]({{ site.url }}/img/doggy-1403.png)
![Higher res doggy woggy]({{ site.url }}/img/doggy-double-rendered.png)

Ok, getting better, although now it looks like some kid scribbled with crayons all over the paper. Oh well, let's move on, keeping in mind that we now can adjust the number of nodes and stroke intensity to adjust the quantity of lines drawn. 

### Manipulating the angle of lines drawn
Compared with [Yamashita's originals](http://www.kumiyamashita.com/constellation/), the average line length in our current portrait is long.  Yamashita generally seems to pass the thread between adjacent nodes rather than across the grid. When we sample for points, however, we're choosing the whole range of points and often get long lines that traverse the picture and make it look "scribbly." Let's change the sampling function to one that chooses from a radius instead:

{% highlight javascript %}
//function drawNextLine
var radius = 2;
result = graph.getValidNodeWithinRadius(previous, thread, radius);

{% endhighlight %}

Here's what we get from sampling from a variable radius:

Radius **1** & **2**:

![Little doggy woggy]({{ site.url }}/img/angle1.png)
![Little doggy woggy]({{ site.url }}/img/angle2.png)

Radius **3** & **5**:

![Little doggy woggy]({{ site.url }}/img/angle3.png)
![Little doggy woggy]({{ site.url }}/img/angle5.png)

**Maximum radius** (sampling only the points that are farthest away) & **random radius** (weighted to a power curve so lower values appear much more frequently)

![Little doggy woggy]({{ site.url }}/img/anglemax.png)
![Little doggy woggy]({{ site.url }}/img/anglerandom.png)

It turns out that in our square grid of nails, changing the sampling radius changes the angle of all possible values that can be drawn. In a radius of 1, for example, the algorithm only chooses points from the surrounding 9, which results in only straight lines and 45 degree angles. This looks kind of mosaic-like. In the max radius, you get hundreds of slightly different angles that make the dog look kind of spooky or phantasmic. It seems like somewhere in the middle is a nice balance between 1) long & short lines and 2) angle consistency and diversity.

### Increasing line connectivity

One thing you might notice about the above lines is that they are often unconnected. In other words, often times the algorithm will fail to find a valid next point and start a new random thread instead of continuing the old one. This results in many one-off lines and abrupt endings, which impact the clarity of the image and make it harder to draw.

Take a look at the above renderings and see if you can figure out why this happens. It's very subtle. (Hint: try looking for the brightest points in the images, which are the ones drawn the most frequently).

It turns out the brightest points are the nodes themselves. If 5 lines share a node and are drawn with intensity of red 30, the node will have intensity of red 150 whereas the surrounding pixels will have an intensity of 30.  What this means for our algorithm is that nodes are peaking out and returning "false" early. This forces our algorithm to begin a totally new line even if many more lines might be able to be drawn from that node. 

This is easy to solve. We just change our drawing function so that it only paints pixels _between_ nodes. The result changes instantly:

**Left**: Same settings with the "random radius" function.  **Right**: Weighted random radius and original (sparse) grid.

![Little doggy woggy]({{ site.url }}/img/connected-dense.png)
![Little doggy woggy]({{ site.url }}/img/connected-sparse.png)

Sweet! This tiny change makes the algorithm effectively fill out most of the edges of the graph and give it a very smooth appearance. We can even use our smaller grid from before and still have a very cohesive portrait of the dog. 

Another cool thing is that this small fix sped up the algorithm considerably as well--these render in 2-3 seconds as opposed to the ~minute it was taking before!

### In the next post:

There's still some work left to do. 

- Adjust from RGB (additive) to real life subtractive color schemes (i.e. CMYK)

- Model color intensity through area averages rather than pixel intensity.

- Physically construct the canvas

[Go to the next post](http://rewonc.github.io/update/2014/11/06/A-thousand-nails-and-three-threads-2.html)





