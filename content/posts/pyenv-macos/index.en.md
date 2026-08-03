---
title: "Managing Python Versions on macOS with pyenv"
date: 2026-07-23T13:30:00-03:00
draft: false
tags: ["Python", "pyenv", "macOS", "Terminal"]
categories: ["Python"]
---

macOS ships with Python, but using the system Python for your projects is asking for trouble. **pyenv** solves this elegantly: install as many versions as you want and choose which one to use per project, without conflicts.

### Installation

```console
brew install pyenv
```

Then add the following to your shell config (`~/.zshrc` or `~/.bashrc`):

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Reload your shell:

```console
source ~/.zshrc
```

### Installing Python versions

List available versions:

```console
pyenv install --list
```

Install the one you need:

```console
pyenv install 3.12.10
```

### Global version

Sets the default Python version for the entire system:

```console
pyenv global 3.12.10
```

### Local version (per project)

This is pyenv's killer feature. Inside a project folder, set a specific version:

```console
cd ~/projects/fooocus
pyenv local 3.10.14
```

This creates a `.python-version` file in the directory. From that point on, any terminal opened in that folder automatically uses that version, no activation needed.

```console
python --version
# Python 3.10.14
```

### Why not just use brew's Python?

`brew upgrade` can update Python and break existing venvs. With pyenv, each version stays isolated — you control when (and if) you upgrade.

For simple projects that always use the latest version, brew works fine. But if you work with tools that require a specific version, `pyenv local` saves a lot of time.
