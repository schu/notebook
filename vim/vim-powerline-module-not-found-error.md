# Vim: `ModuleNotFoundError: No module named 'powerline'`

I use `powerline-status` and `powerline-gitstatus` with Vim and sometimes
encounter the following error when the default Python version diverges from the
one that Vim is linking:

```
Error detected while processing [...]/.vim/vimrc:
line   53:
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'powerline'
line   54:
Traceback (most recent call last):
  File "<string>", line 1, in <module>
NameError: name 'powerline_setup' is not defined
line   55:
Traceback (most recent call last):
  File "<string>", line 1, in <module>
NameError: name 'powerline_setup' is not defined
Press ENTER or type command to continue
```

The problem is that `powerline-status` is not installed in the linked Python
environment:

```
vim --version
[...]
-python            +python3
[...]
Linking: [...] -L/opt/homebrew/opt/python@3.12/Frameworks/Python.framework/Versions/3.12/lib/python3.12/config-3.12-darwin -lpython3.12
```

```
python3 --version
Python 3.11.7
```

For a Homebrew installation of Vim, installing `powerline-status` system-wide
is an option:

```
sudo pip3.12 install --break-system-packages powerline-status powerline-gitstatus
```

`--break-system-packages` is necessary as pip doesn't allow system-wide installations
for externally managed environments by default. In the case of powerline I assume
no risk of breaking my Python installation or OS and skip the error.

```
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try brew install
    xyz, where xyz is the package you are trying to
    install.

    If you wish to install a non-brew-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip.

    If you wish to install a non-brew packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
```

For virtual environments, falling back to system site packages is an option
(if falling back is acceptable in the given context), for example:

```
python3.11 -m venv --system-site-packages venv
```

## Appendix: powerline vimrc setup

```
# vimrc
# [...]
python3 from powerline.vim import setup as powerline_setup
python3 powerline_setup()
python3 del powerline_setup
```

