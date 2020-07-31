---
layout: default
title: This site
permalink: /technical/
---

### Style and templates

This site uses as base the [hacker] theme, with layout templates based both on
the [hacker] and the [minima] themes. 

The pages of the post list and of the posts themselves are modification of the
[minima] home and post layouts. On top of the usual
[metadata](https://jekyllrb.com/docs/frontmatter), the ``post.html`` layout
recognises the ``acknowledgments`` keyword. If found must contain a list of
strings, also in markdown syntax, and renders the elements in a
"Acknowledgments" section between the title and the content. E.g. the following

    acknowledgments: 
        - '[Lucia
          Klarmann](https://www.uva.nl/en/profile/k/l/l.a.klarmann/l.a.klarmann.html),
          for helping me organize my babbling and supervisioning the first draft
          of this post'
        - '[Emre Aydin](eaydin), for pushing me into writing this post'

is rendered as in [this post]({{ site.baseurl }}{% post_url
2017-05-11-virtualenv_anaconda %}).

[hacker]: https://github.com/pages-themes/hacker
[minima]: https://github.com/jekyll/minima
