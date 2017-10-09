
Package multiversion


Imagine that there is numpy v1.10 installed system-wide (e.g. in '/usr/lib64/python3.6/site-packages') but you need or want the latest version of it. You can do it installing it in the user directories, '~/.local/lib/python3.6/site-packages', with the command


    pip install --user --upgrade numpy.


However this newer version will shadow the system version making it hard (althought not impossible) to use it.


Given the fact that python imports the first matching package found in the ``sys.path`` it is non trivial to have multiple versions of the same package working at the same time.
Two typical example are:


* you develop a package and want to be able to use the development version when updating it and the stable version for normal operations
* you use two libraries that requires two different versions of the same dependence


Virtualenvs to the rescue


Virtual environments are a solution to those problems. They create (more or less) isolated environments where the python path is manipulated in order to consider only packages installed within the environment.


This functionality is provided by a number of packages, one of which is virtualenv. Starting with python 3.3. a similar package has been added to the standard library (venv).


After installing virtualenv (instructions here), it is possible to create a new environment with the command:


    virtualenv /path/to/my_vent


It creates a directory “/path/to/my_vent”, containing a number of directories needed to run python. In particular it creates a bin directory containing, among others, python and pip executables. The python is a copy of the system one used when creating the virtual environment. It is possible to use other python interpreters using the ``--python`` option.


Once created, the virtual environment can be activated running


    source /path/to/my_vent/bin/activate


From now on, ``python`` calls the version in the virtual environment, whose path is:


    ['',
    '/path/to/my_vent/lib/python36.zip',
    '/path/to/my_vent//lib64/python3.6',
    '/path/to/my_vent//lib64/python3.6/lib-dynload',
    '/usr/lib64/python3.6',
    '/usr/lib/python3.6',
    '/path/to/my_vent/lib/python3.6/site-packages']


As you can see, the environment is isolated from the rest of the system.
If necessary, it is possible to use system installed packages from within the virtual environment using the ``--system-site-packages`` option when creating it.


You can deactivate the virtual environment using the ``deactivate`` command.


Since each environment is a directory, it is possible to create as many of them as necessary. And being isolated entities, each one can have different packages installed and/or different versions of the same package(s).
Managing virtualenvs and switching between them can easily become tedious, but there are packages, like virtualenvwrapper, that help managing multiple virtualenvs and also execute extra code when creating or activating/deactivating them.


``virtualenvwrapper`` provides a number of commands, for example, to create new environments (mkvirtualenv), list and switch between them (workon) and remove them (rmvirtualenv). Many more commands are available and documented here.
But I need more/other python versions!


One disadvantage of virtual environments, as described above, is the fact that it is possible to create them only for versions of python already installed on the system.


It is of course possible to install new python versions by hand, however there are a couple of drawbacks:


1. it’s up to you;
2. by default python directory structure supports multiple versions, like 3.5 and 3.6; however it is harder to have bugfix releases of the same version, e.g. 3.6.0 and 3.6.1.




If you need other version of python, there are various options:


* Pyenv
* Anaconda


Installing by hand can be instructive, but is hardly something that you want to do very often. and we don’t cover this here.


Pyenv


Pyenv is a suite of shell commands that allow to install and manage multiple python versions and to easily switch between them. All this without shadowing the python versions installed on you system. It is available only for Linux and MacOS X.


Instructions to install pyenv are avaliable at the project github page. Alternatively, an installer is available. Beside being easier, the installer has also the advantage that it installs a number of plugins, for example the one that provide virtualenv integration. The installation does not require special permissions, as all the necessary files are put, by default, in ``$HOME/.pyenv``.


Once ready, you can install your favourite python interpreter with the command


    pyenv install <version>


This will create a new directory inside the pyenv home, compile the python interpreter for you and add all the necessary bells and twistles.
When writing, it is possible to install 308 interpreters:


* CPython: 76 versions, starting with 2.1.3
* PyPy: 117 versions
* Anaconda/Miniconda: 40/36 versions
* jython, ironpy, pyston, stackless: a few of each


Afterwards, you can switch to you new python version using the command:


    pyenv shell <version>


To switch back to the system version(s) use:


    pyenv shell --unset


To know which version are you using issue:


    pyenv version


while:


    pyenv versions


shows all the installed versions.


There are a few pyenv plugins in the wild and one of them integrates it with virtualenv.
Thanks to it, it is possible to create a new virtualenv using:


    pyenv virtualenv <version> <venv_name>


It is of course possible to create a virtualenv using the one of the system python version using the following command:


    pyenv virtualenv --python `which python` <venv_name>


and then to switch to it using


    pyenv shell <venv_name>




Anaconda


Anaconda is a software distribution centerd around python. It comes with a package manager, conda, that allows to install packages as well as create environment similar to the virtualenv described above.

