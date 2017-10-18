---
layout: post
title:  "Python virtualenv and Anaconda"
date:   2017-05-11
categories: python virtualenv anaconda pyastro17
author: Francesco Montesano
acknowledgments: 
    - '[Emre Aydin](eaydin), for pushing me into writing this post'
    - '[Lucia
       Klarmann](https://www.uva.nl/en/profile/k/l/l.a.klarmann/l.a.klarmann.html),
       for helping me organize my babbling and supervisioning the first draft of
       this post'
---

## Python import resolution

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

## Package multiversion

Imagine that there is ``numpy`` v1.10 installed system-wide (e.g. in
``/usr/lib64/python3.6/site-packages``) but you need or want the latest version
of it. You can do it installing it in the user directories,
``~/.local/lib/python3.6/site-packages``, with the command

    pip install --user --upgrade numpy.

However this newer version will shadow the system version making it hard
(although not impossible) to use it.

Given the fact that python imports the first matching package found in the
``sys.path`` it is non trivial to have multiple versions of the same package
working at the same time. Two typical examples are:

* you develop a package and want to be able to use the development version when
  updating it and the stable version for normal operations;
* you use two libraries that requires two different versions of the same
  dependence.

## Virtualenvs to the rescue

Virtual environments are a solution to those problems. They are (more or
less) isolated environments with functionalities to manipulate the python path
on activation or deactivation.

This functionality is provided by a number of packages, one of which is
[``virtualenv``](https://virtualenv.pypa.io/). Starting with python 3.3 a
similar package has been added to the standard library, under the
[``venv``](https://docs.python.org/3/tutorial/venv.html ) name.

After installing ``virtualenv`` following [these
instructions](https://virtualenv.pypa.io/en/stable/installation/), one can
proceed to create a new environment with the command:

    virtualenv /path/to/my_vent

This creates a directory tree under ``/path/to/my_vent``:

    /path/to/my_venv
    ├── bin
    ├── include
    │   └── python3.6m -> /usr/include/python3.6m
    ├── lib
    │   └── python3.6
    │       ├── collections -> /usr/lib64/python3.6/collections
    │       ├── config-3.6m-x86_64-linux-gnu -> /usr/lib64/python3.6/config-3.6m-x86_64-linux-gnu
    │       ├── distutils
    │       ├── encodings -> /usr/lib64/python3.6/encodings
    │       ├── importlib -> /usr/lib64/python3.6/importlib
    │       ├── lib-dynload -> /usr/lib64/python3.6/lib-dynload
    │       └── site-packages
    ├── lib64 -> lib
    └── share
        └── man
            └── man1

The ``bin`` directory contains the ``python`` and ``pip`` programs as well as
the executables installed by other packages. When activating the environment the
``bin`` directory is put in front of the ``PATH``. The other directories reflect
the standard python hierarchy, a number of them being links to the corresponding
system directories necessary to run the python interpreter.

The version of python to use when creating a new virtualenv can be specified
using the ``-p/--python`` option.

Once created, the virtual environment can be activated running

    source /path/to/my_vent/bin/activate

From now on, ``python`` calls the version in the virtual environment, whose
associated path is:

    ['',
    '/path/to/my_vent/lib/python36.zip',
    '/path/to/my_vent/lib64/python3.6',
    '/path/to/my_vent/lib64/python3.6/lib-dynload',
    '/usr/lib64/python3.6',
    '/usr/lib/python3.6',
    '/path/to/my_vent/lib/python3.6/site-packages']

As you can see, the environment is isolated from the rest of the system. If
necessary, it is possible to access system installed packages from within the
virtual environment using the ``--system-site-packages`` swithc when creating
it.

Virtual environments can be deactivated using the ``deactivate`` command.

Since each environment is a directory, it is possible to create as many of them
as necessary. And being isolated entities, each one can have different packages
installed and/or different versions of the same package(s).

Managing virtualenvs and switching between them can easily become tedious.
Thankfully there are packages, like
[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/), that help
managing multiple virtualenvs as well as execute extra code when creating or
activating/deactivating them.

``virtualenvwrapper`` provides a number of commands, for example, to create new
environments
([``mkvirtualenv``](https://virtualenvwrapper.readthedocs.io/en/latest/command_ref.html#mkvirtualenv)),
list and switch between them
([``workon``](https://virtualenvwrapper.readthedocs.io/en/latest/command_ref.html#workon))
and remove them
([``rmvirtualenv``](https://virtualenvwrapper.readthedocs.io/en/latest/command_ref.html#rmvirtualenv)).

## But I need more/other python versions!

One disadvantage of virtual environments is that it is possible to create them
only for versions of python already installed on the system.

It is of course possible to install new python versions by hand, however there
are a couple of drawbacks:

1. it’s up to you;
2. by default python directory structure supports multiple versions, like 3.5
   and 3.6; however it is harder to have bugfix releases of the same version,
   e.g. 3.6.0 and 3.6.1.

Installing by hand can be instructive, but is hardly something that you want to
do very often. And we don’t cover this here.

Fortunately if you need other version of python, there are various options:

* [Pyenv](https://github.com/pyenv/pyenv)
* [Anaconda](https://www.continuum.io/anaconda-overview)

### Pyenv

Pyenv is a suite of shell commands that install and manage multiple python
versions. All this without shadowing the python versions installed on you
system. It is available only for Linux and MacOS X.

Pyenv can be easily installed using an handy
[installer](https://github.com/pyenv/pyenv-installer) or following the
instructions in the [project github page](https://github.com/pyenv/pyenv).
Beside being easier, the installer has also the advantage that it installs a
number of plugins, for example one that provides virtualenv integration. The
installation does not require special permissions, as all the necessary files
are put, by default, in ``$HOME/.pyenv``.

Once ready, you can install your favourite python interpreter with the command

    pyenv install <version>

This will create a new directory inside the pyenv home, compile the python
interpreter for you and add all the necessary functionalities. At 18.10.2017,
it is possible to install 340 interpreters:

* CPython: 83 versions, starting with 2.1.3
* PyPy: 129 versions
* Anaconda/Miniconda: 44/42 versions
* ironpython, jython, micropython, pyston, stackless: a few of each

Afterwards, you can switch to you new python version using the command:

    pyenv shell <version>

To switch back to the system version use:

    pyenv shell --unset

To know which version are you using issue:

    pyenv version

while:

    pyenv versions

shows all the installed versions.

There are a few pyenv plugins in the wild and one of them integrates it with
virtualenv.  Thanks to it, it is possible to create a new virtualenv using:

    pyenv virtualenv <version> <venv_name>

It is of course possible to create a virtualenv using the one of the system python version using the following command:

    pyenv virtualenv --python /usr/bin/python2 <venv_name>

and then to switch to it using

    pyenv shell <venv_name>

### Anaconda

Anaconda is a software distribution centered around python. It comes with a
package manager, conda, that allows to install packages as well as create
environment similar to the virtualenv described above.
