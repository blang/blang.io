+++
date = "2016-04-14T10:51:49+01:00"
title = "Basic python environment"
+++

A short summary on how to setup a good python environment for beginners.

vim support
===========

See [Excellent blog post](https://realpython.com/blog/python/vim-and-python-a-match-made-in-heaven/).

PEP8 Checking
=============
[PEP8](https://www.python.org/dev/peps/pep-0008/) is the style guide for python.
You can enable style checking in vim:

First install [flake8](https://pypi.python.org/pypi/flake8/)

```
pip install flake8
```

Then install [vim-flake8](https://github.com/nvie/vim-flake8)

To check syntax on file save, add this to your vimrc:

```
autocmd BufWritePost *.py call Flake8()
```

Virtualenv
==========

Virtualenv keeps your python package path clean only including project requirements.

```
# Install virtualenv, use pip -V to determine python2 or python3
pip install virtualenv

cd project

# Create virtualenv
virtualenv -p /usr/bin/python2.7 venv

# Activate
source venv/bin/activate

# Install dependencies
pip install nltk

# Add venv to your gitignore
echo "venv/" >> .gitignore

# Freeze your dependencies
pip freeze > ./requirements.txt

# Install dependencies (e.g. after git pull)
pip install -r ./requirements.txt

# Deactivate venv
deactivate
```

Also have a [look at the docs](http://docs.python-guide.org/en/latest/dev/virtualenvs/).

