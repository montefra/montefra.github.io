---
layout: post
title:  "Python virtualenv and Anaconda"
categories: python virtualenv anaconda pyastro17
author: Francesco Montesano
acknowledgments: 
    - '[Lucia
       Klarmann](https://www.uva.nl/en/profile/k/l/l.a.klarmann/l.a.klarmann.html),
       for helping me organize my babbling and supervisioning the first draft of
       this post'
    - '[Emre Aydin](eaydin), for pushing me into writing this post'
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

However this newer version will shadow the system version, making it hard
(although not impossible) to use it.

Given the fact that python imports the first matching package found in the
``sys.path``, it is non trivial to have multiple versions of the same package
working at the same time. 

> "Why would I want to do it?"

Well, think about those two cases:

* You develop and use a package: as you develop it, you need to be able to test
  it; on the other hand you likely want a stable version for your business or
  research. So either you keep switching and reinstalling your package (and it
  is as bad as it sounds) or you install the two versions in two different
  places and switch between the two.
* You use two libraries that requires two different versions of the same
  dependence <sup id="a1">[1](#f1)</sup>.

## Virtualenvs to the rescue

Virtual environments are a solution to those problems. They are (more or
less) isolated environments with functionalities to manipulate the python path
on activation or deactivation.
This functionality is provided by a number of packages: the most used is
[``virtualenv``](https://virtualenv.pypa.io/). Starting with python 3.3 a
similar package has been added to the standard library, under the
[``venv``](https://docs.python.org/3/tutorial/venv.html ) name.

After installing ``virtualenv`` following [the
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
virtual environment using the ``--system-site-packages`` switch when creating
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

> What if I need a version of python that is not installed on my system that
> my package manager does not provides?

Unfortunately virtual environments can be built only for existing python
versions; they cannot be made out of thin air.

The hard way is to install a new python version by hand. But there are might be
a couple of issues:

1. it’s up to you;
2. by default python directory structure supports multiple versions, like 3.5
   and 3.6, but not multiple bugfix releases of the same version, e.g. 3.6.0 and
   3.6.1.

Installing by hand can be instructive, but is hardly something that you want to
do very often <sup id="a2">[2](#f2)</sup>.

Fortunately if you need other version of python, there are options. Here I will
introduce:

* [Pyenv](https://github.com/pyenv/pyenv)
* [Anaconda](https://www.continuum.io/anaconda-overview)

Likely there are other solutions that either I ignore or that have never used.

### Pyenv

``pyenv`` is a suite of shell commands to install and manage multiple
independent python versions. It is available only for Linux and MacOS X.
``pyenv`` can be easily installed using a handy
[installer](https://github.com/pyenv/pyenv-installer) or following the
instructions in the [project github page](https://github.com/pyenv/pyenv) <sup
id="a3">[3](#f3)</sup>. The installation does not require special permissions,
as all the necessary files are put, by default, in ``$HOME/.pyenv``.

Once ready, you can install your favourite python interpreter with the command

    pyenv install <version>

This creates the new directory ``$HOME/.pyenv/versions/<version>``, compiles
the python interpreter for you and adds all the necessary functionalities. As of
18.10.2017, it is possible to install 340 different interpreters:

* ``CPython``: 83 versions, starting with 2.1.3
* ``PyPy``: 129 versions
* ``Anaconda``/``Miniconda``: 44/42 versions
* ``ironpython``, ``jython``, ``micropython``, ``pyston``, ``stackless``: a few
  of each

The command 

    pyenv shell <version>

activates the ``<version>``, while 

    pyenv shell --unset

deactivates it, going back to the default. By default the (sic.) default python
version is ``system``, i.e. typically the one that comes with your operating
system. However it is possible to customise the default python version both
globally, with ``pyenv global <version>``, and locally, with 
``pyenv local <version>``. The latter command creates a file called
``.python-version`` in the directory from where it is executed. When ``cd``-ing
in the directory the ``<version>`` is automatically activated and when leaving
it is deactivated.

The list of all installed version can be obtained with the 

    pyenv versions

command, while the current version is returned by:

    pyenv version

``pyenv`` comes with a number of plugins that expands the basic functionality. A
very interesting one is
[``pyenv-virtualenv``](https://github.com/yyuu/pyenv-virtualenv.git) that
integrates ``virtualenv`` with ``pyenv``. A new virtual environment can be
created with one of the following commands:

    pyenv virtualenv <version> <venv_name>

or

    pyenv virtualenv --python /usr/bin/python3 <venv_name>

The only difference is that the former is based on a ``pyenv`` installed python
version, while the latter uses the system python interpreter.

As for the standard version, Virtual environments can be activated and
deactivate with

    pyenv shell {<venv_name>|--unset}

### Conda

There are some situations that ``pyenv`` and virtual environments cannot solve.
One example that comes to my mind is the graphical library ``Qt`` and its python
wrapper ``PyQt``. Since ``Qt`` is a ``C++`` library, it cannot be compiled with
pip and stored inside a virtual environment. In cases like this, you can use the
[``conda``](https://conda.io/docs/index.html) package and environment manager.
``conda`` is part of the [Anaconda](https://www.anaconda.com/what-is-anaconda/)
software distribution as well as of its lightweight sibling
[Miniconda](https://conda.io/miniconda.html). They can be installed downloading
a script from one of the above website and running it.  Alternatively, if you
already have [pyenv](#pyenv) installed, you can use ``pyvenv install`` to
install any of the 40 Anaconda or Miniconda versions.

Once you have ``conda`` installed, you can use the following commands to
install, update or remove a package:

    conda install <package_name>
    conda update <package_name>
    conda remove <package_name>

If a python package that is not available on the ``conda`` repositories, you can
of install install ``pip`` and use it to install the missing package:

    conda install pip
    pip install <other_package>

As already written, ``conda`` is also an environment manager: it can be used
create/manipulate/delete isolated environments. The command

    conda create -n <env_name> [<package_name1> [<packagename2> [...]]]

creates a new environment called ``myenv`` and installs the provided packages.
``<package_name(n)>`` can be any of the packages available to ``conda``,
including python. As example:

    conda create -n myenv python=3.5

creates an environment called ``myenv`` that uses python 3.5.

Once created, the environment can be activated using:

    source activate <env_name>

and deactivated with:

    source deactivate

The ``activate`` script is located in the Anaconda/Miniconda ``bin`` directory,
alongside ``conda``: if this directory is not in your ``PATH`` you might need to
provide its full name. Once active you can install/update packages as shown
above either using ``conda`` or ``pip``.

Although the ``conda`` environments feels like the python virtualenvs, there are
important differences. As far as I know, the two main ones are:

* The ``conda`` environments are **completely** isolated, and contain their own
  version of the python interpreter. On the other hand ``virtualenvs`` are build
  from an existing python version and, thank to symbolic links, inherit from it
  the standard library. This means that a virtual environment might break if the
  reference python version is removed or updated.
* While ``virtualenvs`` are pure **python** environments, ``conda`` ones handle
  also non-python packages. As example imagine that your system has ``Qt``
  version 5.9.x, but you need to develop an application against an older
  version. You can create a new environment specifying a different version:

    conda create -n qt5_old python=3.6 qt==5.6.0

  and switch to it <sup id="a4">[4](#f4)</sup>.

For more information about differences between ``conda`` and the
``pip``/``virtualenvs`` combo, you can read this [interesting post by Jake
VanderPlas](https://jakevdp.github.io/blog/2016/08/25/conda-myths-and-misconceptions/).
I have used this post for some of the information shown here.

``conda`` has also a few drawbacks. Two I aware of are:

* it is not compatible with [``tox``](https://tox.readthedocs.io/en/latest/):
  ``tox`` leverages virtual environments (and optionally also
  [``pyenv``](https://pypi.python.org/pypi/tox-pyenv)) to check package
  installation and to run test suites for multiple python versions with a simple
  command.
* ``Anaconda`` and ``Miniconda`` are too eager to take ownership of the python
  environment: by default, when the installation is finished, they prepend their
  own bin directory to the system ``PATH``. Because of this, there have been
  reports of [conda libraries like libstdc++ shadowing system
  ones](https://github.com/ilastik/ilastik-build-conda/issues/24) or the conda
  python causing crashes on [desktop
  environments](https://conda.io/docs/user-guide/troubleshooting.html#programs-fail-due-to-invoking-conda-python-instead-of-system-python).

## The End

Thank you for reading this. I hope that someone will one day find this useful.
But most important:

**Have fun!**

## Footnotes

<b id="f1">1</b>&bull; If you need the two library to work at the same time you
are out of luck, sorry. On the other hand, if at least one of the library is
open source, might be a great opportunity to contribute to it. [↩](#a1)

<b id="f2">2</b>&bull; And we don’t cover this here. [↩](#a2)

<b id="f3">3</b>&bull; I suggest to use the installer: it is easier and by
default installs a few useful plugins, for example the one that provides virtualenv
integration. [↩](#a3)

<b id="f4">4</b>&bull; I don't know how many versions of packages like ``Qt``
are supported by [Anaconda](https://www.anaconda.com/), so this might not be as
easy at it sounds. However it might be possible to have more flexibility using
additional repositories or use tools like
[conda-forge](https://conda-forge.org/) to build your own package. [↩](#a4)

