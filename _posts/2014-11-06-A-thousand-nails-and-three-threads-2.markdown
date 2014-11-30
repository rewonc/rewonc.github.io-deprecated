---
layout: post
title:  "A thousand nails and three threads: part two"
date:   2014-11-06 012:05:06
categories: update
---

In the [last post](http://rewonc.github.io/update/2014/11/03/A-thousand-nails-and-three-threads.html) I covered my first attempt at coming up with a representation of an image drawn using strings and nails, in the manner of [Kumi Yamashita's artwork](http://www.kumiyamashita.com/constellation/). This is what I came up with, using a simple algorithm that just chose random points to connect on a square grid of "nails":

![Little doggy woggy]({{ site.url }}/img/connected-sparse.png)

So this looks kind of neat, but isn't ready to be converted to a canvas just yet. The most significant problem is that the color of threads in real life do not combine like the colors on a screen. The RGB color scale is _additive_, which usually is used for the LED's in your screen because red, green, and blue light firing at 100% creates white light. But in real life we're using threads, which when combined in close quarters actually block out white light and appear grayish black. It's _subtractive_, in other words, and the most common subtractive scale is CMYK. This is what laserjet printers use. 

##Switching to a subtractive color scheme

Like printing, for our project we should assume that the background is white and that each line layered on top will subtract from that light. We can do this by writing an adapter between RGB and CMYK values and inserting it in the layer between our image file and our processing algorithms. Then, we will change our algorithm to write lines in CMYK instead of RGB. 

{% highlight javascript %}

Helpers.RGBtoCMYK = function(obj){
  var red = obj.red/255,
    green = obj.green/255,
     blue = obj.blue/255,  
        k = 1 - Math.max(red, green, blue),
        c, m, y;
  if (k === 1) {
    return {c: 0, m: 0, y: 0, k: 1};
  } else{
    c = (1 - red - k) / (1 - k);
    m = (1 - green - k) / (1 - k);
    y = (1 - blue - k) / (1 - k);
    return {c: c, m: m, y: y, k: k};
  }
};

Helpers.CMYKtoRGB = function(obj){
  return {
    red: 255 * (1-obj.c) * (1-obj.k),
    green: 255 * (1-obj.m) * (1-obj.k),
    blue: 255 * (1-obj.y) * (1-obj.k),
  };
};

{% endhighlight %}

Here's the result. Changed the photo to one with a light background so it draws the foreground.

![Little doggy woggy]({{ site.url }}/img/dogCMYK.png)

This is a dog again, if you can't tell. This isn't that impressive, but at least it's drawing colors that we can recreate in real life now.  

##Switching to area averages for estimating darkness

Another huge problem with our model is that in real life, colors don't add by stacking on each other. Two red lines stacked on top of each other don't become a darker red---they stay the same color. In our model, however, they do stack on each other and become darker. Pixels that are painted multiple times turn black.

Compare this with [Yamashita's artwork](http://www.kumiyamashita.com/constellation/). She creates the appearance of darker areas by grouping lines closer together, and the appearance of lightness by having a sparse distribution of lines. We need to do the same in our algorithm. 

We can do this programmatically by averaging the color density in an area, then only allowing a certain number of lines to be drawn in that area. [Check the repo for implementation details](https://github.com/rewonc/nailsandthread). Ditching the animal theme, here's what we get when we apply this methodology to a famous human:

![Clint]({{ site.url }}/img/clint-wide.png)

Did you guess Clint Eastwood? If so, hooray! In this version, the colors overwrite each other and do not add to a different color (like in real life). Instead, we generate darker areas by drawing more lines in that area. Thus, this portrait is made entirely of 4 base colors (cyan, yellow, magenta, and key (black)).  Yet, if you blur your vision or look at the image from afar, Clint's face looks flesh colored and his hair kind of silvery white. Neato.

##Optimizing for connected lines


##Dynamic adjustment of nodes










