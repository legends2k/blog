+++
title = "Blog – fit and finish"
description = "quirks and hacks in making this"
date = "2016-08-12T17:00:46+05:30"
tags = ["tech", "hugo", "typesetting", "tools"]
mathjax = true
+++

I did quite a few things to get this blog up and running the way you see it now.

# Steps I Still Remember

1. Install [Hugo][]; just extract and put in `PATH` --- can't be simpler.
2. Create a dummy site using [the Quickstart guide][Quickstart]; straight forward.
3. Install _[Ghostwriter][]_ inside the `themes` directory.

Hugo recommends that customizations be done outside the theme's directory tree to clearly differentiate customizations.  I followed it.

[Hugo]: https://gohugo.io
[Quickstart]: http://gohugo.io/overview/quickstart/
[Ghostwriter]: http://themes.gohugo.io/theme/ghostwriter/

## 1. Syntax Highlighting

This theme uses *[highlight.js][]* for syntax highlighting while I like *[Pygments][]* since the latter means no processing at the client's end; everything needed is preprocessed and only the final rendering is done at the reader's end.  Even after switching to Pygments, it didn't work as expected with this theme's stylesheet.  Debugged the site with Firefox and made a copula hacks to get it working; see `/static/dist/style.css` for the hacks with comments.  For Pygments, make sure `Python` and Pygments (a `pip` install) are installed and is reachable from the command-line.

Also, prefer using the Hugo _[shortcode][]_ `{{ <highlight> }}` instead of <code>```</code> since it can additionally highlight specific line(s).  The default of Pygments is *Monokai* as opposed to the dull, light background theme that comes with Ghostwriter.

[highlight.js]: https://highlightjs.org/
[Pygments]: http://pygments.org/
[shortcode]: https://gohugo.io/extras/shortcodes/

## 2. Nice Font
I really like the font used in *[The Book of Shaders][]* and saw that the font name is **Baskerville**.  I recalled that for custom web fonts, _reveal.js_ used to do this using [Google Fonts][]; luckily it had _Libre Baskerville_, an open version of the face.  Thankfully the site also gives the code to use it and everything started looking great!  Beautiful fonts everywhere! 😁

[The Book of Shaders]: https://thebookofshaders.com/
[Google Fonts]: http://fonts.google.com

## 3. External Content
Showing any image is no big deal; the usual markdown syntax for showing images would do. It took some time to figure out where to actually place the image file.  The `content` directory is for posts, while the `static` directory is for putting any kind of file or external content that may be referred to by the posts.  Plunking the `.svg`s here and referring them in the post did the trick.  Likewise to show a HTML5 canvas demo, just adding a `.html` and `.js` and embedding the HTML in an `<iframe>` worked.  To remove the extraneous scroll bars I'd to refer [WebGL Fundamentals][] to check out how *Gregg Tavares* did it.

[WebGL Fundamentals]: http://webglfundamentals.org/

## 4. Beautiful Math Equations
Be it for showing inline math, such as `$\pi$`, or full-blown equations like

$$
\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}
$$

_MathJax_ is the go-to solution in browsers.  The _display math_ (MathJax lingo to display an equation in its own line with `\$\$…\$\$`) option works with no quirks; _inline math_ (`\$…\$`) doesn't work properly by default; I'd to [stuff][hugo_markdown_quirk] to get it working.  I added a custom parameter in the post's front matter, which is later used in a partial HTML template to include the [necessary script tag][hugo_markdown] for the post's generated HTML.  MathJax may work-up the `<code>` tags, and the workaround for this is shown in the documentation.  The `TeX-AMS-MML_HTMLorMML` configuration is used, but this should be tested out in all browsers since I remember this getting rendered appallingly in Firefox for the [_2D Transforms 101_][transforms_101] presentation; I'd to resort to `TeX-AMS-MML_SVG` in its case.

[hugo_markdown_quirk]: http://github.com/spf13/hugo/issues/1666#issuecomment-225316394
[hugo_markdown]: http://gohugo.io/tutorials/mathjax/
[transforms_101]: http://legends2k.github.io/2d-transforms-101/

## 5. Fractions and Footnote return text
Things like footnote return link text can be configured directly in the site's `config.toml`.  More such options are explained in the configuration page of Hugo's excellent documentation.  When a fraction is typed out ordinarily like `355/113`, it still shows up like 355/113, which is nice; it does it just with `<sup>` and `<sub>` tags, no CSS or MathJax.  This is due to _Blackfriday_, the underlying Go markdown rendering engine used by Hugo.

## 6. Test and Deploy
Running Hugo with `hugo server` serves the content on-the-fly at `localhost:1313`; port can be changed.  When testing is complete and all is well, the final run would be just `hugo` without any parameters; this generates the entire site tree inside the `public` directory which needs to be hosted.
