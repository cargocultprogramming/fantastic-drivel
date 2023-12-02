---
layout: post
title: Code highlighting in jekyll
description: Highlight code in jekyll and mark individual lines
tags: jekyll ruby liquid
categories: 
featured: true
---
Code highlighting is a standard feature in jekyll. There are plenty of examples and explanations out there, for example from the [Jun711 blog](https://jun711.github.io/web/how-to-highlight-code-on-a-Jekyll-site-syntax-highlighting/) and many other sources. I wont repeat the basic functionality here. This is how this looks for python:

{% highlight python linenos %}

#!/usr/bin/env python
# syntax highlighting

class Bar(object):  # Class definition
    def __enter__(self):
        pass

    def __exit__(self, *args):
        pass

    def foo():  # function definition
        if 'string' in list:
            return []

{% endhighlight %}

I do want to talk about highlighting *individual lines* within code blocks: Jekyll uses the [rouge highlighter](https://github.com/rouge-ruby/rouge) which has the option to mark individual lines to point out individual code parts.

That however is not implemented in jekyll itself, so while the underlying engine has the option jekyll doesnt tie into it. It took me a while and a few unsucessful workaround-attempts and digging in github comments to finally find out that the necessary parsing has finally been added and will be published with the 4.4.0 release of jekyll as noted in [this section of the jekyll docs](https://jekyllrb.com/docs/liquid/tags/#marking-specific-lines).

However this will not allow to use this feature in markdown code blocks, only using the liquid markup. So the full feature will look something like this:

{% raw %}
{% highlight python linenos mark_lines="3 4 5" %}  <br/> code goes here <br/> {% endhighlight %}
{% endraw %}

And - as soon as I have migrated to jekyll 4.4.0, the code below should show some highlighted lines:

{% highlight python linenos mark_lines="1 4" %}

#!/usr/bin/env python                       <-- should be marked
# syntax highlighting

class Bar(object):  # Class definition      # <-- should be marked
    def __enter__(self):
        pass

    def __exit__(self, *args):
        pass

    def foo():  # function definition
        if 'string' in list:
            return []

{% endhighlight %}