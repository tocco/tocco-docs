OpenShift Basics
================


Introduction to OpenShift
-------------------------

`OpenShift`_ provides a management tool to deploy `Docker`_ containers.

Here's what's important:

* A single Docker Image is deployed via continuous delivery. It contains Nice, Java and all runtime
  dependencies. Thus, Nice can be run in any OpenShift/Docker environment.
* OpenShift orchestrates the Docker images, service, routes, etc. and ensures they stay available.

Here are a few resources that help you understand the core concepts:

* `High level architecture of OpenShift <https://docs.openshift.org/latest/architecture/index.html>`_.
* `An introduction to Docker <https://en.wikipedia.org/wiki/Docker_(software)>`_.

Have a look at the full `OpenShift documentation`_ and `Docker documentation`_ if you want to know all the details.

.. _Docker: https://www.docker.com/
.. _Docker documentation: https://docs.docker.com/
.. _OpenShift: https://www.openshift.org/
.. _OpenShift documentation: https://docs.openshift.org/latest/


OpenShift Structure used for Nice
---------------------------------

Every installation consists of the following OpenShift elements:

+------------------------------+----------------------------------------------------------------------------+---------------------------------+
| :term:`deployment config`    | ``nice2-${installation}``                                                  | ``solr-${installation}``        |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+
| :term:`container             | ``nice``                          | ``nginx``                              | ``solr``                        |
| <container (openshift)>`     |                                   |                                        |                                 |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+
| :term:`image stream`         | ``nice-${customer}``              | ``nginx`` [#f4]_                       | ``solr`` [#f5]_                 |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+
| :term:`image stream tag`     | ``test`` or ``production`` [#f1]_ | ``stable`` or ``latest`` [#f2]_        | ``stable`` or ``latest`` [#f2]_ |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+
| :term:`exposed port`         | tcp/8080                          | tcp/8081                               | tcp/8983                        |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+
| :term:`service`              |                                   | ``nice-${installation}``               | ``solr-${installation}``        |
|                              |                                   | (tcp/80 redirected to tcp/8081)        | (tcp/8983)                      |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+
| :term:`route`                |                                   | ``nice-${installation}``               |                                 |
|                              |                                   | (https\://${installation}.tocco.ch)    |                                 |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+
| :term:`persistent volume     |                                   |                                        | ``solr-${installation}`` [#f3]_ |
| claim`                       |                                   |                                        |                                 |
+------------------------------+-----------------------------------+----------------------------------------+---------------------------------+


.. rubric:: Footnotes

.. [#f1] Production systems use the ``production`` tag and test systems the ``test`` tag.
.. [#f2] By default the ``stable`` tag is used. ``latest`` is the staging area and is only deployed on selected systems.
.. [#f3] Mounted at ``/persist`` and only the subdirectory ``/persist/index_data`` is currently used for the :term:`Solr` index.
.. [#f4] Image source is hosted on `Github <https://github.com/tocco/openshift-nginx>`_ and the ``latest`` tag is
         automatically built on `Dockerhub <https://hub.docker.com/r/toccoag/openshift-nginx/>`__.
.. [#f5] Image source is hosted on `Github <https://github.com/tocco/openshift-solr>`_ and the ``latest`` tag is
         automatically built on `Dockerhub <https://hub.docker.com/r/toccoag/openshift-solr/>`__.
