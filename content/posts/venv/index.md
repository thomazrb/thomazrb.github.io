---
title: "Virtual Environments in Python"
date: 2023-05-01T14:59:20-03:00
draft: false
tags: ["Virtual Environment", "venv", "Python"]
categories: ["Python"]
ShowToc: true
---

## Virtual environments

Python has become one of the most popular programming languages in the world, thanks to its simplicity, versatility, and powerful libraries. One of the key features that make Python such a great language to work with is its ability to create virtual environments. Virtual environments are isolated Python environments that allow you to install and manage packages and dependencies without affecting other projects or your system's global Python installation.

In this post, we'll walk through the process of creating and managing virtual environments in Python using "venv".

There are several environment managers in Python, but we will focus on "venv", as it is very simple, it is already installed in Python 3 and is the manager recommended by the [Python Documentation](https://docs.python.org/3/library/venv.html).

## Creating a virtual environment

To create a virtual environment with "venv", access your project page, and type the command:

```console
python -m venv .venv
```
It will create a new virtual environment in the current directory with the name ".venv". The "python" command is used to invoke the Python interpreter and the "-m" option is used to run a specified module as a script. In this case, we're running the "venv" module, as I said before, which is included with Python and provides a built-in way to create virtual environments. The dot (".") before the directory name specifies that the virtual environment will be a hidden directory (in Mac and Linux). The name of the virtual environment, in this case, ".venv", can be changed to any other valid directory name.

The installed packages will be stored inside this directory and it is important to remember to add this directory to the ".gitignore" file if you use Git.

## Activating a virtual environment

To activate the virtual environment, you must use the following command:

**Windows:**
```console
.venv\Scripts\Activate.ps1
```
**Mac and Linux:**
```console
source .venv/bin/activate
```
It is important to notice that when you do that, you will see the virtual environment name before the prompt between parenthesis, in this case:

```console
(.venv) %
```
You will install all necessary packages and run your scripts as long as the virtual environment is active. Any Python packages you install or update will be confined to this virtual environment.

## Deactivating a virtual environment
After that, you could deactivate it with the command:

```console
deactivate
```

The absence of the virtual environment name from the shell prompt indicates that the session has returned to its default state and the virtual environment was deactivated.

## Final considerations
Always you need to work on your project, you will activate it again.

Remember to create a virtual environment for each project you have. All of them can have the same name because they are in different directories. The advantage of always using the ".venv" name is that most online documentation is named that way, and your ".gitignore" file is usually already configured to ignore the ".venv" directory.




