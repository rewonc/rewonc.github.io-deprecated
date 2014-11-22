---
layout: post
title:  "A thousand nails and three threads: part two"
date:   2014-11-06 012:05:06
categories: update
---

In the [last post](http://rewonc.github.io/update/2014/11/03/A-thousand-nails-and-three-threads.html) I covered my first attempt at coming up with a representation of an image drawn using strings and nails, in the manner of [Kumi Yamashita's artwork](http://www.kumiyamashita.com/constellation/). This is what I came up with, using a simple algorithm that just chose random points to connect on a square grid of "nails":

![Little doggy woggy]({{ site.url }}/img/connected-sparse.png)

So this looks kind of neat, but isn't ready to be converted to a canvas just yet. The most significant problem is that the color of threads in real life do not combine like the colors on a screen. The RGB color scale is __additive__, which usually is used for the LED's in your screen because red, green, and blue light firing at 100% creates white light. But in real life we're using threads, which when combined in close quarters actually block out white light and appear grayish black. It's _subtractive_, in other words, and the most common subtractive scale is CMYK. This is what laserjet printers use. 

Like printing, for realizing the portrait we should assume that the background is white and that each line layered on top will subtract from that light. We can do this by writing an adapter between RGB and CMYK values and inserting it in the layer between our image file and our processing algorithms. Then, we will change our algorithm to write lines in CMYK instead of RGB. We should get an inversely drawn image--in the case of the dog, many lines in the black space around the dog and less lines in the body of the dog itself (as it is white). 

{% highlight javascript %}

var 

{% endhighlight %}

Here's the result:



