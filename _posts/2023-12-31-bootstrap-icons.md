---
layout: post
title: Use boostrap icons on a website
description: Boostrap icons with npm
tags: frontend web-development bootstrap
categories: 
featured: true
---
There are plenty of icon libraries out there to use with website development. Popular choices are [feather icons](https://feathericons.com/) and [fontawesome](https://fontawesome.com/search) which I used up until now.

Feather icons is open source but only offers around 300 icons. This is a good choice for smaller projects, that only need a few icons.

Fontawesome offers 30000 icons (not sure how they count variants) but only a small subset of those is free.

I stumbled over the new (2023) [icon set from bootstrap](https://icons.getbootstrap.com/?q=foo) which seems to offer a reasonable middle ground by offering a growing list of (currently) 2000 icons, all of which are available as font and svg variants. Here I go over how to use them.

The [bootstrap icon website](https://icons.getbootstrap.com/) isn't very starter friendly unfortunately, since the usage section is hidden below the long list of icons and once there, it presents the most uncommon, verbose usage options first and only later the classic usage with the icon tag. Once you get over that, it's great and straight-forward to use.

Alternatively, bootstrap icons can be accessed via the [bootstrap icons github repository](https://github.com/twbs/icons).

To serve the icons over my own server rather than relying on a CDN (also this allows direct use of svg icons in js, editing icons etc.) install them via npm in your project.

```bash
npm i --save bootstrap-icons
```

Then I can include the bootstrap-icons stylesheet on my website in the main bootstrap scss file, for example in `site-bootstrap.scss`.

```scss
@import "~bootstrap-icons/font/bootstrap-icons.css";
```

Of course this step requires some sort of scss pre-processing like in a webpack pipeline or like it is automatically included in jekyll.

Once the scss preprocessor has run, you can use the icons in the usual manner:

```html
<i class="bi-alarm" style="font-size: 2rem; color: cornflowerblue;"></i>
```
