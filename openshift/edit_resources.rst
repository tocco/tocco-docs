Edit resources
==============

Deployment Config
-----------------

You can modify the config using ``oc edit dc ${INSTALLATION}``.

Adjust Memory / CPU
^^^^^^^^^^^^^^^^^^^

You can change the resource for any of the containers. Here is how it looks like for Nice:

.. code:: yaml

   containers:
   - name: nice
     resources:
       limits:
         cpu: "1"
         memory: 2Gi
       requests:
         cpu: 100m
         memory: 1Gi

The **requests** section describes the guaranteed resources and **limits** the maximum available. Both are optional. The
OpenShift documentation has some more details on `requests and limits`_.

.. _Requests and Limits: https://docs.openshift.org/latest/admin_guide/overcommit.html#requests-and-limits


.. _java-and-nice-params:

Scale Up/Down
^^^^^^^^^^^^^

.. code:: yaml

    spec:
        replicas: 1

``replicas`` is the number of simultaneously running instances.

You can also use this command to scale Nice instances:

.. code::

    oc scale dc/nice-${CUSTOMER} --replicas=${N}

This scales ``CUSTOMER`` to ``N`` replicas. Use 0 to stop all instances.

Setting Java and Nice Parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``env`` section can be used to change Java Paramaters, application.properties and hikaricp.properties.

The env section looks something like this:

.. code:: yaml

    containers:
      - name: nice
        env:
        - name: NICE2_HIKARI_dataSource__databaseName
          value: nice_pege
        - name: NICE2_HIKARI_dataSource__serverName
          value: postgresqlssd
        - name: NICE2_HIKARI_dataSource__user
          value: nice_pege
        - name: NICE2_JAVA_OPT____Dch__tocco__nice2__runenv
          value: production
        - name: NICE2_APP_nice2__enterprisesearch__solrUrl
          value: http://solr-pege:8983/solr/nice2_index


The following environment variables are supported:

===================  ===================================================================================================
NICE2_APP_*          Add custom entries to ``application.local.properties``.
NICE2_HIKARI_*       Add custom entries to ``hikiricp.local.properties``.
NICE2_JAVA_PARAM_*   Pass custom parameters to Java.
NICE2_NICE_ARG_*     Pass custom argument to Nice.
FLUENTD_TARGET_HOST  Target host for :term:`fluentd`. (Fluentd forwarding is enabled automatically if this variable is
                     set)
FLUENTD_TARGET_PORT  Target port for :term:`fluentd`. (defaults to 24224)
===================  ===================================================================================================

.. important::

    OpenShift does not currently allow ``.`` (period) or ``-`` (hyphen) to appear as key of an environment variable.
    [#f2]_

    As workaround:

        ==============  ===========================
        instead of      use
        ==============  ===========================
        ``.`` (period)  ``__`` (double underscore)
        ``-`` (hyphen)  ``___`` (triple underscore)
        ==============  ===========================

    For instance, instead of:
        ``NICE2_JAVA_OPT_-Dch.tocco.nice2.runenv=production``
    use:
        ``NICE2_JAVA_OPT____Dch__tocco__nice2__runenv=production`` [#f1]_

Examples
````````

    Adding entries to application.local.properties:
        Expected entry:
            ``nice2.web.core.compressJavascript=true``

        Environment variable:
            ``NICE2_APP_nice2.web.core.compressJavascript=true``

    Adding entries to hikaricp.local.properties
        Expected entries:
            ``dataSource.databaseName=nice2_dockertest``
            ``dataSource.password=``
            ``dataSource.serverName=172.17.1.11``

        Environment variables:
            ``NICE2_HIKARI_dataSource.databaseName=nice2_dockertest``
            ``NICE2_HIKARI_dataSource.password=``
            ``NICE2_HIKARI_dataSource.serverName=172.17.1.11``

    Setting Java options:
        Expected options passed to java(1):
            ``-Xmx1g``
            ``-Dch.tocco.nice2.runenv=production``

        Environment variables:
            ``NICE2_JAVA_OPT_-Xmx1g=``
            ``NICE2_JAVA_OPT_-Dch.tocco.nice2.runenv=production``

    Setting Nice arguments:
        Expected arguments passed to ch.tocco.nice2.boot.Nice2
            ``-logConfig=/app/etc/custom_logback.xml``

        Environment variable:
            ``NICE2_NICE_ARG_-logConfig=/app/etc/custom_logback.xml``


.. rubric:: Footnotes

.. [#f1] Replacement is done from right to left, preferring the longest possible replacement. Replacing only the three
         rightmost underscores in a quadruple underscore.
.. [#f2] https://github.com/openshift/origin/issues/8771
