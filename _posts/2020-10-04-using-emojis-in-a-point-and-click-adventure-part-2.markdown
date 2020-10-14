---
layout: post
title:  "Using Emojis in a Point and Click Adventure - Part 2"
date:   2020-10-04 17:10:19 +0100
categories: blog js13k
---

In this post, I will be looking at the Emoji cross platform and cross browser issues I faced in creating my [js13kgames](https://js13kgames.com/) entry. I hadn't been working long on ["The King's Missing Page"](https://2020.js13kgames.com/entries/the-kings-missing-page) when I realised just how different the various emoji font files were across the different platforms and browsers.

I also learnt that the frequency at which different operating systems updated their emoji fonts to support new Unicode versions varied, so that some platforms are more likely to be 3 or 4 years behind the standard, whereas others are completely up to date.

## The js13kgames emoji rule

As this year's contest was starting, I became aware of a js13kgames rule that I hadn't previously realised was there. It reads as follows:

> "you are allowed to ask users to live-load a web font to support some characters or emoji on devices that can't display them properly, but you have to make sure your game will work without them"

This intrigued me a bit. The [Twemoji](https://github.com/twitter/twemoji) font was one such font that could be used. Others include [Noto Color Emoji](https://github.com/googlefonts/noto-emoji), [OpenMoji](https://openmoji.org/), and perhaps [JoyPixels](https://www.joypixels.com/) (formerly [EmojiOne](https://github.com/eosrei/emojione-color-font)). The above rule was a bit vague on what was allowed though. Could I *always* load and use the font? That would make the experience consistent across platforms, but it felt a little bit like cheating. The rule states that "you have to make sure your game will work without them".

I think it is clear from the wording of the rule that you need to make an effort first to use the emoji font that is available to the platform that the game is running on. One solution would be to pop up a message asking the user to choose between the platform specific font or the live loaded font. I couldn't help feeling like this would distract from the game itself and draw attention to the fact that an external resource was being used.

## Detecting the supported emoji unicode version

I decided instead to implement an automatic solution, one that would detect whether the platform supported the necessary emoji characters, and only if it didn't would it "live load" the chosen fallback font. This felt like it was in the spirit of the rules.

The plan I had for my game initially depended on emojis up to and including unicode 12 being available, so I needed a way of detecting whether the platform it was running on supported that version. This seemed a bit tricky at first, but then I discovered that there was a unicode character (\uffff) that would always render the same as an unsupported character (&#xFFFF;). Different platforms render an unsupported emoji character in different ways, but a given platform will render all unsupported emoji characters in the same way.

I was therefore able to render the known unsupported character first and then compare that to the result of attempting to render the Unicode 12 character. If they matched, then Unicode 12 was not supported and it would automatically enable the fallback font.

{% highlight javascript %}
/**
 * Detects what Emoji Unicode version is available by default.
 */
static detectEmojiVersion() {
    // The game requires at least Unicode 12 to work without Twemoji. We 
    // work out whether it supports Unicode 12 by rendering a known 
    // unsupported char and then compare it to what is rendered for a 
    // Drop of Blood, which is part of Unicode 12.
    let NO_EMOJI = Util.renderEmoji('\uffff', 100, 100).toDataURL();
    if (Util.renderEmoji('🩸', 100, 100).toDataURL() == NO_EMOJI)) {
        Util.twemoji = true;
        document.body.classList.add('twemoji');
    }
}
{% endhighlight %}

With that class added to the body, the game can then dynamically load the chosen fallback font. If there is a @font-face declaration block for the font, the browser will not automatically load it. But if one of the CSS selector blocks becomes active, perhaps in response to the "twemoji" class being added to the DOM in the example above, then the browser will download the font.

{% highlight css %}
@font-face { 
  font-family: "twemoji";
  src: url('https://lanceewing.github.io/twemoji.ttf');
}
/* The browser will only download the twemoji font if the twemoji class is present */
.twemoji #controls {
  font-family: "twemoji";
}
{% endhighlight %}

By using a technique like this, the font is only downloaded when the platform the game is running on doesn't support the necessary emoji version.

In the example above, we used Twemoji as the fallback font. As mentioned earlier, there are several other options we could have gone with and perhaps Twemoji wasn't necessarily the best one. In order to make the best choice, I had to learn more about the various available fonts and formats, and their pros and cons.

## Emoji fonts and formats

The automatic fallback mechanism described above is just that: a fallback mechanism. The normal case would hopefully be when the platform already supports the required unicode version. So let's have a look at those "standard/default" fonts first, alongside some of the other alternatives.

The following image shows how a single emoji character appears differently depending on what font is used (images sourced from [Emojipedia](https://emojipedia.org/house-with-garden/)):

![image info](/images/different-emoji-fonts.png)

The examples on the left are the default emoji fonts on particular operating systems, whereas those on the right are not associated with any particular OS. These eight are not the only Emoji fonts in use, but at the time of writing, they are probably the main ones that are still under active development. Some are available for free use, some for a fee, and others are fully proprietary and restricted solely to certain operating systems, apps or devices.

| Provider | Description |
| ----------- | ----------- |
| Apple     | Known as "Apple Color Emoji". Used on Apple Macs, iPhones and iPads. Not available otherwise. |
| Google    | Known as "Noto Color Emoji". Used on most Android devices. Comes by default on many Linux distributions. Available for free use. |
| Microsoft | Known as "Segoe UI Emoji". Used only on Windows machines. Not available otherwise. |
| Samsung   | Used on Samsung Android devices. Not available otherwise. |
| Twitter   | Known as "Twemoji". Used on Twitter. Available for free use. |
| Facebook  | Used on Facebook. Not available otherwise. |
| JoyPixels | Low-res version available for free use. Paid version at higher res. Formerly called EmojiOne. |
| OpenMoji  | Free, open source Emoji font. |

Games submitted for the js13kgames competition are likely to encounter only the following emoji fonts: Segoe UI Emoji (Windows), Apple Color Emoji (Mac/iPhone/iPad), Noto Color Emoji (Android/Linux), and Samsung.

As we can see in the image above, the same emoji looks quite different across those four emoji fonts. The Apple, Noto, and Samsung fonts are all bitmap fonts that look like hand painted pictures. The Windows font, on the other hand, is a vector font with layers of flat colours, and has the thick black outline that was discussed at length in [part 1](/blog/js13k/2020/09/27/using-emojis-in-a-point-and-click-adventure-part-1.html) of this series of articles.

These are not the only differences. Each vendor has drawn the house image itself in a different way. A different number of windows. Some with a chimney, some without. Even when they have the same features, those features don't always align in the same places. These differences mean that a game can't rely too much on what the emoji looks like. They have to be treated in a more generic way.

## Multi-color Glyphs

In addition to a different appearance, there are several standards and formats for how Color Emojis are stored in a font file. Microsoft, Google, Apple, and Mozilla/Adobe all came up with different formats, and unfortunately they're all being used.

They all do essentially the same thing, which is to extend the standard font file formats to include color image data, but they do this in different ways.

| Format | Description |
| ----------- | ----------- |
| COLR/CPAL | Created by Microsoft. Uses standard font glyphs, but with multiple layers of different colors. As a consequence, uses flat colors (i.e. no shading). Vector format (not SVG though), so scales well. Surprisingly quite widely supported. Examples: "Segoe UI Emoji", "EmojiOneMozilla", "TwemojiMozilla" |
| CBDT/CBLC | Created by Google. Replaces standard font glyphs with PNG bitmap format. Doesn't scale well. Larger font sizes will blur pixels. Example: "Noto Color Emoji" |
| SBIX | Created by Apple. Emoji data stored as PNG bitmap data, so also doesn't scale well. Not used outside the Apple ecosystem. Example: "Apple Color Emoji" |
| OpenType-SVG | Created by Mozilla and Adobe. Supports Vector and Bitmap formats. |

I realised after learning about these different formats that whichever "fallback" font I used, it would need to be in a format that worked on as many platforms and browsers as possible. All four formats are now incorporated into the OpenType spec, but support varies across the different browsers and platforms.

SBIX was ruled out early on, since only Apple was using it, and their font was not freely available. SBIX is supported by Safari, Chrome and Edge, but not Firefox. Even though the format is supported in several browsers, it is only useful on Apple devices.

## OpenType-SVG

OpenType-SVG was ruled out next, due to [Chrome's refusal](https://bugs.chromium.org/p/chromium/issues/detail?id=306078) to support it, mainly due to performance concerns. Now that Edge is using Chromium, Edge also no longer supports OpenType-SVG.

If you read up on OpenType-SVG, you'll see that it was at one time mentioned as being well positioned to become the golden standard. It doesn't seem like this is how things have panned out, in fact with Chromium having no plans to implement it, and Edge now using Chrominum, the browser support has actually regressed in recent times.

The following website is a good one for testing OpenType-SVG support on your browser:

[http://xerographer.github.io/reinebow/](http://xerographer.github.io/reinebow/)

I tried the above web page while writing this article and can confirm that it was only in Firefox that the color fonts were displayed.

The following table shows what I believe the current state to be. It is a modified version of the table that appears in Ollie Williams' article ["It All Started With Emoji: Color Typography on the Web"](https://css-tricks.com/it-all-started-with-emoji-color-typography-on-the-web/) on CSS-Tricks. All I've changed is the OpenType-SVG in Edge, which is no longer available:

<table style="text-align: center;">
  <tbody>
    <tr><td></td><th scope="col">Chrome</th><th scope="col">Safari</th><th scope="col">Edge</th><th scope="col">Firefox</th></tr>
    <tr><th scope="row">SVG-in-Opentype</th><td>❌</td><td>❌</td><td>❌</td><td>✅</td></tr>
    <tr><th scope="row">COLR/CPAL</th><td>✅</td><td>✅</td><td>✅</td><td>✅</td></tr>
    <tr><th scope="row">SBIX</th><td>✅</td><td>✅</td><td>✅</td><td>❌</td></tr>
    <tr><th scope="row">CBDT/CBLC</th><td>✅</td><td>❌</td><td>✅</td><td>❌</td></tr>
  </tbody>
</table>

## CBDT/CBLC

CBDT/CBLC didn't have consistent support across all browsers either. I couldn't make Noto Color Emoji load using @font-face in Firefox no matter what I tried. By default, the sanitizer in Firefox rejects CBDT/CBLC, but there is a preference (gfx.downloadable_fonts.keep_color_bitmaps) to turn on support. Unfortunately Noto Color Emoji characters still do not show even after enabling this preference.

I was also a bit skeptical about how good a bitmap format emoji font would look like when drawn at large pixel sizes, such as 400px. Would it look too blurry? The following is an example of using Noto Color Emoji:

![image info](/images/noto_castle_scene.png)

The castle in the above image appears quite blurry. Although not so noticeable on a mobile device, it becomes quite obvious on a desktop monitor. The bitmap image for the emoji has been stretched bigger than it is designed to be. 

A vector format wouldn't suffer from this. 

## COLR/CPAL

What did that leave me with? COLR/CPAL. It appeared to be supported by all browsers, and had been for some time. Microsoft had been quite cleaver in creating a format that utilised the glyph format already used in font files. It wasn't as flexible as SVG, but it was a vector format and so did scale well. The only problem was that the Windows "Segoe UI Emoji" format was the only COLR/CPAL font that I'd heard of so far.

I couldn't use the Windows emoji font as the fallback font. Technically though, I probably could, and I did think about it for a bit, but it wasn't the right approach, because Microsoft didn't allow this, so would violate the licensing section of the js13kgames rules.

Most of the free to use open source emoji fonts were not in the COLR/CPAL format. But after a bit of searching, I discovered that Mozilla had created COLR/CPAL versions of a couple of them.

## EmojiOne Mozlla Font

The first font they did this with was the EmojiOne font. Firefox used to ship with a COLR/CPAL version of EmojiOne, but Mozilla replaced this with with a COLR/CPAL version of the Twemoji font back in April 2018, in response to [EmojiOne no longer being open source](https://bugzilla.mozilla.org/show_bug.cgi?id=1358240).

For the same reason, I couldn't really use the latest EmojiOne font (i.e. JoyPixels) either. Perhaps I could use the COLR/CPAL version that Mozilla produced before the EmojiOne licensing changed, since EmojiOne v2 was bound by an opensource license and therefore Mozilla's conversion of v2 would also be free to use. The drawback with that though was that it would limit the game to Unicode 9, which would miss out on the past few years of new emoji characters.

## Twemoji COLR font

The [Twemoji COLR/CPAL font](https://github.com/mozilla/twemoji-colr), on the other hand, appeared to be perfect for my needs. It ticked all the boxes and it looked good as well:

![image info](/images/twemoji_game_screen.png)

So this is what I choose to use as my fallback font for most of the time I was working on my 2020 js13kgames entry.

## Cross Platform Testing

I did some testing on Windows, Macbook, Android and a Samsung phone. Windows 10 was using Unicode 12. After a little research, it appeared that Windows kept its emoji font up to date through regular Windows updates. 

The Android and Samsung phones, assuming they were on the latest Android release, also were on Unicode 12. The following video is a full playthrough of "The King's Missing Page" on an Android phone:

<div style="margin-bottom:15px">
  <div style="position:relative;padding-top:56.25%;">
    <iframe src="https://www.youtube.com/embed/_Y4EhzEDbEg?start=110" frameborder="0" allowfullscreen
      style="position:absolute;top:0;left:0;width:100%;height:100%;"></iframe>
  </div>
</div>

What surprised me is that the Apple Macbook Pro I tested on was only at Unicode 9. It seemed that the MacOS was not updating its emojis as regularly as other operating systems were. This was unfortunate, because the Apple emojis were argubably the most visually appealing, despite being a bitmap format and therefore prone to the blurring issue.

![image info](/images/apple_halloween_house.png)

It was possible that there were a lot of Macbooks out there that were also several Unicode versions behind, which would mean that my game would fall back on the Twemoji font rather than showing the native Apple Color Emoji font.

I worried over this for a while, because it felt to me that potentially a rather high proportion of players out there would end up seeing the fallback Twemoji font.

I felt strongly enough about this that I decided to do a stock take of all of the emojis I was using from Unicode versions 10, 11 and 12. Perhaps I could replace those emojis with something similar from earlier Unicode versions, which is exactly what I ended up doing. Before long, the requirement to have Unicode 12 available had been reduced to Unicode 9.

## EmojiOne V2 Revisited

Out of curiosity more than anything, having reduced the required Unicode verson back to verson 9, I wondered what the game would look like with the EmojiOne COLR font. 

![image info](/images/emojione_game_screen.png)

It didn't look too bad, but it did show how emojs from different fonts can be drawn from different angles, which makes placing them on the screen, in a way that works across all possible fonts, a bit tricky. In the following example, the lorry on the left is not meant to be parked on the grass facing out. In all other emoji fonts that I'd seen, this emoji is drawn side on, similar to the vehicle on the right.

![image info](/images/emojione_game_screen_2.png)

## A Reversal

Having been through the process of changing the game to use only Unicode 9 or below, in the end I felt that perhaps the fallback mechanism would no longer be needed, and I could save some precious bytes that could be used elsewhere.

So despite implementing the fallback mechanism described earlier in this article, and choosing a suitable font to use as that fallback, I ended up removing it from the final version of the game. To be honest, there was still a part of me that didn't feel like it was in the spirit of the js13kgame contest to rely on a live loaded font file, so I felt happier that I didn't need it afterall.

After the contest had ended, I received feedback from someone who played my game who said that certain emojis were not showing on his machine. So perhaps even Unicode 9 wasn't old enough to justify removing the fallback mechanism. I had to draw the line somewhere though.

## Takeaways

The following are the key points that I took away from my time researching the various emoji fonts:

* COLR/CPAL currently has the widest support across browsers and platforms.
* The Mozilla Twemoji COLR font is therefore a good fallback font, and is up to date and free to use.
* OpenType-SVG surprisingly has the worst support across the various browsers.
* Apple (SBIX) and Google (CBDT/CBLC) fonts do not scale particularly well at the larger sizes needed to render things like full size buildings in a game.
* If you're not using a "live loaded" fallback font, then using newer emoji characters risks them not being rendered on some people's machines.

## Part 3...

That is all for this part.

In the next part, I'll be looking much closer at how I have used emojis in "The King's Missing Page", focusing primarily on the mouse interaction with the emojis.