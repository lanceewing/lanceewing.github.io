---
layout: post
title:  "Using Emojis in a Point and Click Adventure - Part 1"
date:   2020-09-27 13:47:15 +0100
categories: jekyll update
---

This year I entered the [js13kgames](https://js13kgames.com/) contest for the 6th time. For those who have seen my previous entries, you will have seen that three of them are point and click graphic adventures. I also entered the inaugural gamedev.js jam earlier this year, with what was once again a point and click adventure.

<a href="https://js13kgames.com/games/down-the-drain/index.html">
  <img src="/images/down-the-drain.png" width="49%">
</a>
<a href="https://js13kgames.com/games/back-to-life-adventure/index.html">
  <img src="/images/back-to-life.png" width="49%">
</a>

Since I first heard of the js13kgames game jam back in 2013, I have wondered whether it would be possible to create a point and click adventure game that felt like a full and challenging game. Was 13k enough space to create something like the old Lucasfilm and Sierra On-line classics of the 80s and 90s?

My first two attempts (shown above) fell a little short, mainly because I ran out of time before I ran out of space. But even so I felt that the rooms would all end up looking nearly the same.

## Emojis as an art asset

Earlier this year the solution occurred to me. I had seen games in previous years of the js13kgames contest that had used emoji characters. Looking through the full set of emojis, I could see lots of different buildings, a few different trees, people, animals, and items. Could this be the way to create a graphic adventure game?

![image info](/images/missing-page-black-edges-2.png)

The main problem appeared to be how they are rendered on Windows 10. As shown in the image above, emojis on Windows are rendered with a relatively thick black outline. The bigger you draw the emoji, the thicker the black outline becomes. Buildings, vehicles and trees all have ugly outlines that look out of place.

## Removing the black outline

Emojis are unicode characters rendered using a particular font. The Windows emoji font is called "Segoe UI Emoji". It isn't possible to style that font to remove the thick black outline, because its part of the graphics that make up the colour glyph of the character.

I spent some time playing around on codepen.io to see if could draw an emoji to an HTML canvas and then manipulate the pixels to remove the black outline. The idea I came up with was to apply a pixel fill algorithm starting at an area of the canvas outside of where the emoji is and then customising the fill algorithm to detect pixels that look like they are part of the black outline and to then remove them.

The following Pen shows the final code that I arrived at. You'll need to view this on a Windows machine to  appreciate what it is doing:

<p class="codepen" data-height="407" data-theme-id="dark" data-default-tab="js,result" data-user="lance_ewing" data-slug-hash="jOqgPGo" style="height: 407px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Emoji on Canvas">
  <span>See the Pen <a href="https://codepen.io/lance_ewing/pen/jOqgPGo">
  Emoji on Canvas</a> by Lance Ewing (<a href="https://codepen.io/lance_ewing">@lance_ewing</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

Why use a fill algorithm? This is mainly so that areas of black inside the emoji are not also removed. It is only the black around the edge of the emoji that we're interested in removing. So the fill floods in towards the edges of the emoji and then stops when it gets to the other side of the black outline.

## Detecting the black outline

There is a bit more to it than what I described above. We might naively think that the whole of the black outline is pure black, by which I mean an RGB value of 0,0,0. Due to antialiasing with both the area outside the emoji, and the area within the emoji, the outline takes on various subtle shades and tints. This is what makes the outline look smooth when rendered, but it is a pain to try to remove.

The following code does a reasonable job:

{% highlight javascript %}
// Get the RGBA values for the pixel at the current position.
let [red, green, blue, alpha] = imgData.data.slice(pos, pos + 4);

// Calculate the perceived brightness of the pixel.
let brightness = Math.round(Math.sqrt(
    (red * red * 0.299) + 
    (green * green * 0.587) + 
    (blue * blue * 0.114)
));

let spread = false;
if (alpha == 0) {
    // If the pixel is fully transparent, then let flood fill spread.
    spread = true;
}
else if ((red == 0) && (green == 0) && (blue == 0)) {
    // If the pixel is pure black, then let flood fill spread, and 
    // make pixel fully transparent.
    imgData.data[pos + 3] = 0;
    spread = true;
}
else if (brightness < 70) {
    // If the RGB value of the pixel is below a certain perceived 
    // brightness level, then adjust the alpha channel to match 
    // and let the flood fill spread.
    imgData.data[pos + 3] = Math.round(brightness * (255 / 70));
    spread = true;
}
{% endhighlight %}

The first case, where the alpha channel is zero, is simply the fill flooding into the transparent background parts of the canvas. The second case, where RGB are all zero, is where the pixel is pure black and therefore deemed to be part of the black outline, and so the fill floods into that pixel and it is set to transparent.

If we stopped there, and didn't have the third case, then the resulting image would have a jagged edge. To avoid this, the third case takes advantage of the existing antialiasing between the inside edge of the black outline and the colourful interio of the emoji. It detects this by calculating the perceived brightness of the pixel, and then if it is below a certain threshold, it adjusts the alpha channel of the pixel to match the brightness. This has the effect of making the edge of the emoji look smooth after removing the black outline.

The threshold of 70 was determined by trial and error after testing the code with various Windows emojis.

## The end result

Removing the black outline from the Windows emojis in this way allows more attractive scenes to be created. If we render the same scene shown earlier in this post, then the end results is as follows:

![image info](/images/missing-page-no-edges.png)

Obviously this is only of concern on a Windows machine, but it's likely that many of the participants in the js13kgames contest are using Windows.
