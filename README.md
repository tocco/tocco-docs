# Tocco DevOps Documentation

[![Documentation Status](https://readthedocs.org/projects/tocco-docs/badge/?version=latest)](https://tocco-docs.readthedocs.io/en/latest/?badge=latest)

This documentation is based [Sphinx](http://www.sphinx-doc.org/en/stable/) and
[reStructuredText](www.sphinx-doc.org/en/stable/rest.html) and is automatically built on
[Read the Docs](https://readthedocs.org/projects/tocco-docs/).

## Build Locally

You need tho have this dependency installed:

* python3-sphinx
* python3-sphinx-rtd-theme
* javasphinx (`pip3 install --user javasphinx`)

Now you can build the documentation using:

```
    make html
```

the generated file can be found in `_build/html/`.
