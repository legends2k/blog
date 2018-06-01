+++
tags = ["tech", "tools", "writing"]
date = "2016-07-11T17:00:46+05:30"
description = "static site generators and how they can help"
title = "Hello, Hugo!"
mathjax = true

+++

Many [static site generators][SSG] are coming up these days; 175 and still counting.  Apart from the prime selling point of content creation from the comfort of your favourite text editor in [Markdown][], most have a wide gamut of themes to choose from.  You are free to [remix the theme to your taste][site_customizations] however you see fit.  Want to change the line spacing and paragraph alignment?  Sure thing!  Don't like anything?  Ditch them all and whip up something new.  Share it with others, perhaps?  You do not have to be a web programmer or designer to customize themes; I am a C++ programmer but I had no trouble picking up CSS3; just a few days of tussling did the trick.

You get these advantages in most SSG's but [Hugo][] has one more: it has *no* dependencies from an end-user viewpoint; download and extract it on any machine, and get down to stuff that matters --- content.  I think it gets this by virtue of being authored in [Go][].

This is a test page I tried to check out Hugo; to verify its fitness as a technical blog generator that supports the following sorely-needed scenarios for this web log.

# Text Formatting

* Formatting
    + **bold**
    + *italics*
    + `code snippets`
    + ~~strikeout~~ 
    + blockquotes
    + custom font
    + line spacing
* Footnotes
* Unicode, UTF8 and emoticons support
* Math equations
* Tables
* _SVG_ render with captions
* Interactive Demos with HTML5 Canvas

[Markdown]: https://daringfireball.net/projects/markdown/basics
[Go]: https://golang.org
[SSG]: http://www.staticgen.com/
[Hugo]: https://gohugo.io
[site_customizations]: /note/site_customizations

# Blockquote and Footnote
This is an obscure alternative to the famous "*The quick brown fox…*" pangram[^1]

> Exploring the zoo, we saw every kangaroo jump and quite a few carried babies. 

## Unicode, UTF8 and Special Characters
பிறந்த குழந்தைகட்கு இந்த பாடலைப் பாடிக்காட்டுவது வழ்க்கம்:

**அ**ம்மா இங்கே வா! வா!  
**ஆ**சை முத்தம் தா! தா!  
**இ**லையில் சோறு போட்டு  
**ஈ**யை தூர ஓட்டு!  
**உ**ன்னைப் போன்று நல்லார்  
**ஊ**ரில் யாவர் உள்ளார்?  
**எ**ன்னால் உனக்குத் தொல்லை  
**ஏ**தும் இங்கே இல்லை!  
**ஐ**யம் இன்றி சொல்வேன்  
**ஒ**ற்றுமை என்றும் பலமாம்!  
**ஓ**தும் செயலே நலமாம்!  
**ஔ**வை சொன்ன மொழியாம்!  
அ**ஃ**தே எமக்கு வழியாம்!

ஏனெனில், இப்பாடலில் அனைத்து உயிரெழுத்துக்களும் ஒவ்வொரு அடியின் தொடக்கத்திலும் இடம் பெரும்.

`--` gives an `EN DASH` (--), while `---` is an `EM DASH` (---). Fractions, like 9/22, are beautifully rendered too 😃

## Code

This C++ code is syntax highlighted.  It also has a particular line highlighted; this line prints `hello, world!` on the terminal:

{{< highlight cpp "hl_lines=5" >}}
#include <iostream>

int main()
{
    std::cout << "hello, world!\n";
}
{{< /highlight >}}

## Math Equation
Perhaps this

$$
e^{i\pi}+1=0
$$

equation is more beautiful than the highly over-rated $E = mc^2$.  They are rendered here with MathJax which is supported by Hugo elegantly.

# Table
A table with different alignments:

|Command  | Description |  Help|
|:---          |     :---:      |           ---:|
| `git status`   | Know the current status of the repo | `man git-status`    |
| `git diff`     | Diff tracked but not-yet-staged files | `man git-diff`      |


# Vector Graphics
Let's see how an `SVG` shows up with an accompanying caption!

{{< figure src="/images/hello_hugo/cub_bez_arc.svg" title="Cubic Bézier Circles" >}}

## Interactive Demo

Red is the viewer here; green dots are end points of walls. Click and drag one of them to see where the vision ray from the viewer can go past the walls.

<iframe style="width: 640px; height: 480px; border: 1px solid black; margin-left: auto; margin-right: auto; display: block; box-sizing: border-box;" src="/demos/vision_beyond.html">
<!-- <iframe style="overflow:hidden;width:640px;height:485px" src="/demos/vision_beyond.html" frameborder="0"> -->
</iframe><p></p>

An external HTML document, containing JavaScript, is embedded in an `iframe` here.

[^1]: A sentence using *every* letter of the alphabet at least once.
