---
layout: post
title:  "Markdown and Liquid tags"
date:   2015-07-18 10:03:05
categories: news
---
This is a short reminder for the [Markdown](https://en.wikipedia.org/wiki/Markdown)
tags used in this **Jekyll**-powered website:

Code and syntax highlighting
============================
Code should use the liquid templating tags
{% raw %}
    {% highlight c++ %}
    #include <iostream>

    int main() { // Hello
      std::cout << "Hello world";
    }
    {% endhighlight %}
{% endraw %}

the above snippet will render
{% highlight c++ %}
#include <iostream>

int main() { // Hello
  std::cout << "Hello world";
}
{% endhighlight %}

Inline preformatted text is `also supported` by enclosing it into backticks `

Posts
=====
New posts should be placed in the `_posts` directory and follow the name convention `YYYY-MM-DD-name-of-post.markdown`
(only use .markdown if those are markdown posts).

Links are inserted with the usual markdown syntax `[caption](http://link.com)`

> Quotes are also possible with angle brackets > prepending the sentences

Mathematical formulas
=====================
Mathematical notations and expressions are supported through MathJax

    $$ \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

the above will render

$$ \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

Inline expressions like this: \\( f(x,y) \\) are also supported with the following syntax

    \\( f(x,y) \\)


Rebuilding the website
======================
After modifying the static contents Jekyll will need to rebuild the website. This can be
done with `jekyll build` or by testing the changes out with the integrated webserver
`jekyll serve`


Useful links
============

* [Jekyll docs][jekyll]
* Bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh].
* Questions on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
