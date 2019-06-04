tocco-theme
===========

This package provides styling variables which are consumed by all other packages and enables an uniform look.

Variables
---------

A limited and concise set of variables to define typography, colors, radii and spacing. All adjacent fragments are wrapped in a single object and consumed by ThemeWrapper.

colors
^^^^^^

Variables paper, text and primary are the main colors. Whereas signal colors are used to emphasize certain states.

To produce a good legibility, variable paper must be a light color and variables text and primary must have a high contrast to color paper.

.. code-block:: js

  colors: {
    paper: '#fff',
    primary: '#9E2124',
    text: '#212121',
    signal: {
      danger: {
        paper: '#EF9A9A',
        text: '#B71C1C'
      },
      info: {
        paper: '#81D4FA',
        text: '#0288D1'
      },
      success: {
        paper: '#A5D6A7',
        text: '#1B5E20'
      },
      warning: {
        paper: '#FFF59D',
        text: '#F57F17'
      }
    }
  }

fontFamily
^^^^^^^^^^
Use web safe fonts or import hosted fonts.

Two font stacks are defined. Variable regular contains the primary fonts, whereas variable monospace is consumed by just a few components.

Variable url allows to load fonts from web services. The value must be an uri and is loaded with CSS @import url().

.. code-block:: js

  fontFamily: {
    monospace: 'Menlo, Monaco, Consolas, "Courier New", monospace',
    regular: 'Roboto, "Helvetica Neue", Helvetica, Arial, sans-serif',
    url: 'https://fonts.googleapis.com/css?family=Roboto&display=swap'
  }

fontSize
^^^^^^^^
All font sizes are calculated automatically as an exponential scale. Both values must be provided as unitless numbers. Base is interpreted as rem.

.. code-block:: js

  fontSize: {
    base: 1,
    factor: 2
  }

This example would create a scale of …, 0.25, 0.5, 1, 2, 4, 8, 16, ….

Most text elements use the value of base as rem. Only headings have an increased font size and only a few elements like figcaption have a decreased font size.

fontWeights
^^^^^^^^^^^

Change font weights for extra light or extra bold typography. Ensure that the declared font family does provide such weights. Subsequent code block shows the default.

.. code-block:: js

  fontWeights: {
    regular: 400,
    bold: 700
  }

lineHeights
^^^^^^^^^^^

Line height is a factor applied on font size. Regularly, value regular is used. Only a few components provide a dense mode.

.. code-block:: js

  lineHeights: {
    dense: 1,
    regular: 1.4
  }

radii
^^^^^

Radii is used to round corners of elements like panels, field and buttons.

.. code-block:: js

  radii: {
    regular: '4px'
  }

space
^^^^^

Base and factor are used to control vertical and horizontal white space rhythm. All spaces are calculated automatically as an exponential scale. Both values must be provided as unitless numbers. Base is interpreted as rem.

.. code-block:: js

  space: {
    base: 1.4,
    factor: 2
  }

It is recommended to use a power of two for variable factor to ensure a correct horizontal alignment.


Customizing
-----------

Any theme property can be overwritten. The customized theme must be passed to method createApp of appFactory and is merged into the default theme by ThemeWrapper.

Fourth parameter of createApp is an object. Include the customized theme in that object on path input.customTheme.
