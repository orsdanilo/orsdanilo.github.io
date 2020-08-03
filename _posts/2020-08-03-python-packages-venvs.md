---
title: Python Packages and Virtual Environments
toc: True
toc_label: Table of Contents
categories:
  - Blog
tags:
  - Python
  - Virtual Environment
---

<style>
table:nth-of-type(1) {
    display:table;
    width:100%;
}
</style>

There are several tools to keep Python projects organized and running without compatibility issues. In fact, it can get confusing, as some are similar and seem to do the same thing. This article highlights the main ones, hopefully giving good directions for Python newcomers and eventually clearing some points Python developers may have overlooked. We start with brief clarifications on the concepts of packages and virtual environments and the default tools for Python, then present the `conda` distribution, and conclude with the similarities and differences between `conda` and `pip`/`venv`.

## Python Packages

It is a good practice to refactor code so as to make it more reusable. A Python file containing variables, classes and/or functions can be distributed and reused as a module. Suppose we write the file `foo.py`:
```python
def bar(x, y):
  return x + y
```
This file is a module. The [Python doc on modules](https://docs.python.org/3/tutorial/modules.html) says:
> A module is a file containing Python definitions and statements. The file name is the module name with the suffix .py appended.

Running Python or a Python file from the same folder, the module can be imported in the following manner:
```python
>>> import foo
>>> foo.bar(1,2)
3
```

Or alternatively:
```python
>>> from foo import bar
>>> bar(1,2)
3
```

It is also possible to create shorter aliases for the imports, binding the alias to the module contents. Some famous ones are:
```python
>>> import numpy as np
>>> import pandas as pd
>>> import matplotlib.pyplot as plt
```

We can import all statements from a module using:
```python
>>> from <module> import *
```
However, this is not recommended, as it can be inefficient and possibly hide some things that were already defined (namespace collision).

As the project grows and scales, these modules should be organized in a directory structure containing such types of files, which is then called an import package. We can use to a certain extent the analogy of packages as file system directories, and modules as files. More on packages and imports can be found on [this page](https://docs.python.org/3/reference/import.html). 

As is presented on the docs, a regular package is typically implemented as a directory containing an `__init__.py` file. When it is imported, this `__init__.py` file is implicitly executed, and the objects it defines are bound to names in the package’s namespace. `__init__.py` can be an empty file in the simplest case, or contain initialization code for the package. 

Import packages usually contain multiple files and are not so convenient for distribution. Enter source distribution packages, or *sdists*, which are compressed archives (`.tar.gz` files) containing one or more packages or modules, as is explained [here](https://packaging.python.org/overview/). Python also created the wheel package format (`.whl`), that allows shipping libraries with compiled artifacts, making installation faster.

Distributions can be found in repositories, called Package Indexes, which contain a web interface to automate package discovery and consumption. The default one for the Python community is the [Python Package Index (PyPI)](https://pypi.org). All Python developers can consume and distribute their distributions by making the `.tar.gz` and the `.whl` files available in this repository. Instructions on how to package and upload a distribution to PyPI can be found [here](https://packaging.python.org/tutorials/packaging-projects/).

### pip

[Pip](https://pip.pypa.io/en/stable/) is the recommended installer for Python packages. Packages can be easily installed from PyPI with this command:
```shell
$ pip install <packagename>
```

Here are some other useful commands to upgrade, uninstall and list installed packages, respectively:
```shell
$ pip install --upgrade <packagename>
$ pip uninstall <packagename>
$ pip list
```

## Virtual Environments

Sometimes different Python applications with different dependencies and specific package version requirements need to be executed on the same system. If they were all in the same Python installation, compatibility issues would arise. A solution for this kind of problem is the use of virtual environments.

Quoting the [Python documentation](https://docs.python.org/3/tutorial/venv.html), a virtual environment is:
> A self-contained directory tree that contains a Python installation for a particular version of Python, plus a number of additional packages.

With different virtual environments, the applications can run independently from one another, without conflicting requirements.

### venv

Since version 3.5 of Python, the recommended tool to create virtual environments is [`venv`](https://docs.python.org/3/library/venv.html#module-venv), a built-in Python 3 module. The process is the following: navigate to the folder where the environment should be placed and enter the command: 
```shell
$ python3 -m venv <envname>
```
It will by default use the newest Python 3 version installed on the computer.

To activate the virtual environment on Linux or MacOS, run:
```shell
$ source <envname>/bin/activate
```

Or on Windows:
```shell
$ <envname>\Scripts\activate.bat
```

After running this activation script, `(<envname>)` should be prefixed to the command prompt, showing what is the current environment. To exit the current env, type
```shell
$ deactivate
```

Though it is common practice to create the virtual environment inside the project directory, it is not a good idea to keep it in the project repository, due to OS specifities and folder size. The best alternative is to persist the list of required packages and their versions to a file with the command below:
```shell
(<envname>) $ pip freeze > requirements.txt
```

And then anyone who needs to run the project can obtain the packages in the following manner:
```shell
(<envname>) $ pip install -r requirements.txt
```

## Anaconda

[Anaconda](https://www.anaconda.com) is a free and open-source distribution of the Python and R programming languages for scientific computing and data science. It comes with approximately 250 scientific packages installed out-of-the-box and Anaconda Navigator, a Graphical User Interface (GUI).

If you're not new to Python or programming in general and somewhat confortable with the idea of using the Command Line Interface (CLI), it is preferred to use Miniconda instead, which installs only `conda` (more on that below) and, on Windows, Anaconda Prompt.

### conda

[Conda](https://conda.io/en/latest/) is an open source package management system and environment management system. Like `pip`, it can install Python packages, though it is language agnostic, and can deal with other programming languages and their dependencies as well. Like `venv`, it can create and manage virtual environments, and it can also manage different Python versions, which would require another tool otherwise, such as `pyenv` or `virtualenv`.

Similar to `venv`, a virtual environment can be created with a simple command. Here the environment folder will be created in Anaconda's default folder.
```shell
$ conda create --name <envname>
```

Alternatively, the environment can be created in another directory, e.g. the project folder. We only need to provide the flag `--prefix` alongside the path for the new environment folder:
```shell
$ conda create --prefix <envpath>
```

`python=<pythonversion>` or `<packagename>` can be appended to the `conda create` command, creating an environment with such Python version, and with such package and its dependencies, respectively.

A list of the existing environments can be obtained by typing either one of these:
```shell
$ conda info --envs
$ conda env list
```

Packages can be installed from the conda repository by executing the following:
```shell
(<envname>) $ conda install <packagename>
```

As with `pip`, packages and their dependencies can be exported to a file, only here it is a YAML file:
```shell
(<envname>) $ conda env export > environment.yml
```

And then the file can be used to recreate the environment:
```shell
$ conda env create -f environment.yml
```

Here is a comparison table between `conda` and `pip` taken from the [Anaconda Blog](https://www.anaconda.com/blog/understanding-conda-and-pip):

| | conda	| pip |
|:---:|:---:|:---:|
| Manages | Binaries	| Wheel or source |
| Can require compilers |	No	| Yes |
| Package types |	Any | Python-only |
| Creates environment |	Yes, built-in	| No, requires virtualenv or venv |
| Dependency checks	| Yes | No |
| Package sources | Anaconda repo and cloud | PyPI |

It should be noted that it is not such a fair comparison, as `conda` aims to do more than `pip`, containing more features and not being restricted to Python. 

An important additional piece of information is the number of packages available in each tool: while the Anaconda Repository features around two thousand packages, PyPI boasts more than 250 thousand projects from the Python community. Therefore, it is not uncommon to find with `pip` a library that's not available with `conda`. 

## Conclusion

All in all, `conda` is a great tool for doing data science work. Gathering different functionalities all in on tool certainly makes things a bit more fluid. If you're doing pure Python work, `pip`+`venv` will probably do the job just fine. For general data science projects with varied scientific libraries and dependencies, it may be a good idea to use `conda`. Since the two package managers aren't exactly interchangeable, the recommended approach is to use an isolated `conda` environment, trying to install everything with `conda` and backing off to `pip` when needed.

In this article I tried to gather some information on the available tools for Python development. I recommend that you check the numerous references along this page, do your research and experiments, and then decide what works best for you. I hope it was helpful!