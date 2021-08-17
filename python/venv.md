# VENV Refresher

See more at the docs tutorial [here](https://docs.python.org/3/tutorial/venv.html).

## Bootstrapping

To begin:

```console
> pwd
/Users/rk/workspace/venv-learning
> python3 -m venv my-env
> ls -lah my-env
total 8
drwxr-xr-x   6 rk  staff   192B Aug 17 14:47 .
drwxr-xr-x   4 rk  staff   128B Aug 17 14:47 ..
drwxr-xr-x  13 rk  staff   416B Aug 17 14:47 bin
drwxr-xr-x   2 rk  staff    64B Aug 17 14:47 include
drwxr-xr-x   3 rk  staff    96B Aug 17 14:47 lib
-rw-r--r--   1 rk  staff   111B Aug 17 14:47 pyvenv.cfg
```

This results in a `my-env` directory! This directory is like a snapshot of my code's dependencies:

* Python interpreter
* Standard library
* Other supporting files (like scripts in `/bin`!)

The actual (virtual) environment itself typically lives in a `.venv` directory and must be "activated" from the project root:

Since I'm Unix/MacOS:
```console
> source my-env/bin/activate
```

Note the change in console prompt:

```console
(my-env) >
```

Now, when I run my Python programs, I'll be doing so in a self-contained, version-locked context.

## Using & Managing Dependencies

`pip` is the Python package manager. It installs, updates, removes packages and uses the "Pythong Package Index" ([https://pypi.org](https://pypi.org)) as the default library registry.

Let's say I want to work with a REST API; I could use the standard [`requests`](https://github.com/psf/requests) HTTP library:

```console
(my-env) > python -m pip install requests
```

Now I can see `requests` and its dependencies if I list the following:

```console
(my-env) > ls -lah my-env/lib/python{VERSION}/site-packages
(my-env) > pip list
Package            Version
------------------ ---------
certifi            2021.5.30
charset-normalizer 2.0.4
idna               3.2
pip                21.2.4
requests           2.26.0
setuptools         41.2.0
urllib3            1.26.6
```

To declare this a bit more explicitly, I can use a `requirements.txt ` file:

```console
(my-env) > pip freeze > requirements.txt
(my-env) > cat requirements.txt
certifi==2021.5.30
charset-normalizer==2.0.4
idna==3.2
requests==2.26.0
urllib3==1.26.6
```