Drools
======

Drools is a rule engine which we use in Nice2 to implement customer logic. Unlike logic implemented in Java which can
only be written by our developers, we have the possibility to let anyone write these rules in a running system.

A rule always consist of two parts, the ``when`` and ``then`` section. The ``when`` part is used to check for so called
facts. The existence of facts or their contents can be checked, but no code can be executed, it only checks state.
That's where the ``then`` part comes in. In there it's possible to write Java code and interact with the facts found
in the ``when`` part.

The entire documentation for our version of Drools can be found
`here <https://docs.jboss.org/drools/release/5.3.0.Final/drools-expert-docs/html/ch05.html>`__.

Usages
------

.. toctree::
   :maxdepth: 3

   drools_educationrequirement.rst
   drools_qualification.rst
