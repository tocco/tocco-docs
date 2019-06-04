# Tocco Documentation

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

The generated files can be found in `_build/html/`.

### Live reload while you type

Install `sphinx-autobuild`:
```
pip install --user sphinx-autobuild
```

Then run with:
```
make livehtml
```

And then visit the webpage served at http://127.0.0.1:8000. Each time a change to the documentation source is detected,
the HTML is rebuilt and the page automatically reloaded.


## Customizations

This documentation offers the following extensions to Sphinx/reST:

### `global.rst`

The file [`global.rst`](global.rst) in the root directory is automatically included in all *.rst files. Use it for roles, links, etc. that need to be available globally.

### "strike" Role

Use ``:strike:`some text` `` to strike through ~~some text~~.

### Color Roles

Use ``:blue:`…` ``, ``:green:`…` `` or ``:red:`…` `` to color text.

### "ticket" Role

Use ``:ticket:`OPS-24` `` to reference a ticket [in Jira](https://toccoag.atlassian.net).

### "vshn" Role

Use ``:vshn:`TOCO-125` `` to reference ticket TOCO-125 in [VSHN's issue tracker](https://control.vshn.net/tickets).

### Custom CSS

CSS can be customized in [`_static/css/custom.css`](_static/css/custom.css) as needed.
