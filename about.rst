About this Documentation
========================

This documentation is based `Sphinx`_ and `reStructuredText`_ and is
automatically built on `Read the Docs`_.

Build Locally
-------------

Install Dependencies
^^^^^^^^^^^^^^^^^^^^

You need tho have this dependency installed:

* python3-sphinx
* python3-sphinx-rtd-theme
* graphviz
* python3-sphinx-autobuild (optional, needed for live reload)

On Debian-based Linux, dependencies can be installed like this::

    apt install python3 python3-sphinx-rtd-theme graphviz
    apt install python3-sphinx-autobuild  # optional, needed for live reload


Build the documentation
^^^^^^^^^^^^^^^^^^^^^^^

Now you can build the documentation using::

    make html

The generated files can be found in `_build/html/`.

Live reload while you type::

    make livehtml

And then visit the webpage served at http://127.0.0.1:8000. Each time a change to the documentation
source is detected, the HTML is rebuilt and the page automatically reloaded.


Customizations / Extensions
---------------------------

This documentation offers the following extensions to Sphinx/reST:

* ``global.rst``:

    The file `/global.rst`_ in the root directory is automatically included in
    all \*.rst files. Use it for roles, links, etc. that need to be available
    globally.

* *strike* Role

    source::

      :strike:`some text`

    rendered:

      :strike:`some text`

* Color Roles

   source::

     :blue:`blue text`
     :green:`green text`
     :red:`red text`

   rendered:

     | :blue:`blue text`
     | :green:`green text`
     | :red:`red text`


* *ticket* Role

   Use ``:ticket:`OPS-24``` to reference a ticket *OPS-24* in `Jira`_.

* *vshn* Role

   Use ``:vshn:`TOCO-125``` to reference ticket *TOCO-125* in `VSHN's issue tracker`_.

* Custom CSS

   CSS can be customized in `/_static/css/custom.css`_ as needed.

* Graphviz

   Graphviz can be used to draw graphs using the *dot* language.


   documentation and examples:

     * `examples <https://graphs.grevian.org/example>`_
     * `more examples <https://renenyffenegger.ch/notes/tools/Graphviz/examples/index>`_
     * `node shapes <https://www.graphviz.org/doc/info/shapes.html>`_ (e.g. shape=pentagon)
     * `important attributes <https://graphs.grevian.org/reference>`_
     * `all attributes <https://graphviz.org/doc/info/attrs.html>`_

   .. |graph| replace:: abc

   Example, source::

     .. graphviz::

        digraph {
          label="Sample Graph"

          # define nodes
          start [ shape=house ]
          one
          third [
              label=<neither <font color='red'>one</font><br/>nor <font color='red'>other</font>>,
              shape=diamond
          ]
          # other  # created implicitly through use
          subgraph cluster1 {
            label="Cluster"
            A
            B
          }
          lonely [ URL="https://www.tocco.ch", label="lonely\n(very)" ]
          end [ shape=circle ]

          # force nodes to be of same rank (=displayed at same height)
          { rank=same one other third lonely }

          # define connections
          start -> { one other third }
          one -> end
          third -> one
          third -> end [ penwidth=3.0 ]
          other -> end [ color=blue, label="to the end" ]
          other -> other [ label=back, fontcolor=violet ]

          { A B } -> lonely [ dir=both ]
        }

   Example, rendered:

   .. graphviz::

      digraph {
        label="Sample Graph"

        # define nodes
        start [ shape=house ]
        one
        third [
            label=<neither <font color='red'>one</font><br/>nor <font color='red'>other</font>>,
            shape=diamond
        ]
        # other  # created implicitly through use
        subgraph cluster1 {
          label="Cluster"
          A
          B
        }
        lonely [ URL="https://www.tocco.ch", label="lonely\n(very)" ]
        end [ shape=circle ]

        # force nodes to be of same rank (=displayed at same height)
        { rank=same one other third lonely }

        # define connections
        start -> { one other third }
        one -> end
        third -> one
        third -> end [ penwidth=3.0 ]
        other -> end [ color=blue, label="to the end" ]
        other -> other [ label=back, fontcolor=violet ]

        { A B } -> lonely [ dir=both ]
      }


.. _Sphinx: http://www.sphinx-doc.org/en/stable/
.. _reStructuredText: www.sphinx-doc.org/en/stable/rest.html
.. _Read the Docs: https://readthedocs.org/projects/tocco-docs/
.. _/global.rst: https://github.com/tocco/tocco-docs/blob/master/global.rst
.. _Jira: https://toccoag.atlassian.net
.. _VSHN's issue tracker: https://control.vshn.net/tickets
.. _/_static/css/custom.css: https://github.com/tocco/tocco-docs/blob/master/_static/css/custom.css
