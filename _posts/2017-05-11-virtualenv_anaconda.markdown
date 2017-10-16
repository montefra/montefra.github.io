---
layout: post
title:  "Python virtualenv and Anaconda"
date:   2017-05-11
categories: python virtualenv anaconda pyastro17
author: Francesco Montesano
acknowledgments: 
    - '[Emre Aydin](eaydin) for pushing me into write this post'
    - '[Lucia
       Klarmann](https://www.uva.nl/en/profile/k/l/l.a.klarmann/l.a.klarmann.html)
       for helping me organize my babbling and supervisioning the first draft of
       this article'
---

### Python import resolution

When importing a module, python searches for a file or a package, i.e. a directory
containing a file called ``__init__.py``,  in a number of directories. This list
is stored in the ``sys.path`` variable, and 

    ['',
    '/usr/lib/python36.zip',
    '/usr/lib64/python3.6',
    '/usr/lib64/python3.6/lib-dynload',
    '~/.local/lib/python3.6/site-packages',
    '/usr/lib64/python3.6/site-packages',
    '/usr/lib64/python3.6/_import_failed',
    '/usr/lib/python3.6/site-packages']


If the package is present in, e.g. ``~/.local/lib/python3.6/site-packages``, it
will be imported from there, and the following directories are going to be
ignored.  The first entry of the above list is the current directory, which
allow to import local modules.
