---
layout: post
title: Creative commons letter icons
description: Include creative commons symbols as unicode icons in your website
tags: jekyll css
categories: 
featured: true
---

A lot of websites use the [Creative Commons](https://creativecommons.org/) license(es) to share their creative work. I wanted to do so on this blog, but was confronted with two options I didn't like. I could either show the license as image or as a simple text, like for example [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) or include an [image from the creative commons website](https://creativecommons.org/mission/downloads/).

It's also possible to include the icons as svg files for example from fontawesome or similar icon libraries.

However, I went with another option, by including the minimal webfont from [Daniel Aleksandersen's blog](https://www.ctrl.blog/entry/creative-commons-unicode-fallback-font.html).

I include the font definition in the css definitions for the blog, restricted to the footer, where I want to use them:

{% highlight css %}
@font-face {
  font-display: swap;
  font-family: CCSymbols;
  font-synthesis: none;
  src: url(CCSymbols.woff2) format(woff2),
       url(CCSymbols.woff)  format(woff);
  unicode-range: u+a9, u+229c,
                 u+1f10d-1f10f,
                 u+1f16d-1f16f;
}

footer {font-family: sans-serif, CCSymbols;}
{% endhighlight %}

The tiny font files, that I downloaded from the blog linked above, need to be dropped into `assets/css/` for jekyll.

In the html-footer then, I can use the html notation for the symbols (e.g. from <https://en.wikipedia.org/wiki/Creative_Commons_license>) and the browser will render the symbols in the first webfont it encounters, that support the symbols. In our case this is the `CCSymbols` font. Here is how this looks when modifying the footer of a jekyll page with the al-folio theme.

{% highlight html %}

<footer>
    ...
    <div class="col-12 col-md-3 container">
        <span class="container" style="font-size:20px; vertical-align: middle;"><a href="https://creativecommons.org/licenses/by-nc-sa/4.0/">&#127341;&#127343;&#127247;&#127246;</a></span>
        2023 cargocultprg
    </div>
    ...
</footer>

{% endhighlight %}

That allows me to show the symbols as *normal* text that I can use in a simple link. The css-attribute `vertical-align: middle` ensures that all symbols appear alligned with the other text content.

For the final result take a look at the lower-left of the footer of this website.
