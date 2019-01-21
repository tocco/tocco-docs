Nice Configuration
==================

.. _nice-memory:

Nice Memory
-----------

Adjusting Memory
````````````````

#. Find the current memory settings

    .. parsed-literal::

        $ oc get dc nice -o yaml
        …
        spec:
          …
          strategy:
            …
            resources:             # resources for the :term:`pre-hook pod` (:term:`DB refactoring` is done in this pod)
              requests:
                cpu: "1"
                memory: 1945Mi
            …
          template:
            …
            spec:
              containers:
              - name: nginx        # resources for :term:`Nginx`
                …
                resources:
                  requests:
                    cpu: 30m
                    memory: 20m
                …

              - name: **nice**         # resources for Nice
                …
                resources:
                  **limits**:
                    **memory**: 10Gi
                  **requests**:
                    cpu: "1"
                    **memory**: 2470Mi
                …

#. Adjust it as required

    Adjusting **limits.memory**:

        Unless Nice or another process in the Nice container is killed because of this limit, there is no need to change
        it.

        .. note::

            This limit needs to be set pretty high in order to ensure there is enough memory for :term:`wkhtmltopdf`
            which is started as process in same pod.

    Adjusting **requests.memory**:

        This is used by OpenShift to ensure enough memory is available on the node. It is also used to set the Java
        heap memory (-Xmx…) to a sensible value. Increase this if Nice itself needs more memory.

    .. code-block:: bash

        oc set resources --limits=memory=8Gi --requests=memory=3500Mi dc/nice

    Check out OpenShifts introduction to `Requests and Limits`_ for more details.

Java Heap Memory
^^^^^^^^^^^^^^^^

Java heap memory is automatically set. The minimum value (-Xms…) is hardcoded and the maximum is set based on
**limits.requests** shown above. By default, a hardcoded factor in the OpenShift entrypoint script is used to calculate
the max. memory. Alternatively, the env. variable **MEMORY_FACTOR** can be set (**requests.memory** * **MEMORY_FACTOR**
= **MAX_JAVA_HEAP**).


.. _app-properties-in-openshift:

Setting Java and Nice Parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``env`` section can be used to change Java Parameters, application.properties and hikaricp.properties.

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
          value: http://solr:8983/solr/nice2_index


The following environment variables are supported:

===================  ===================================================================================================
NICE2_APP_*          Add custom entries to ``application.local.properties``.
NICE2_HIKARI_*       Add custom entries to ``hikiricp.local.properties``.
NICE2_JAVA_PARAM_*   Pass custom parameters to Java.
NICE2_NICE_ARG_*     Pass custom argument to Nice. (Not applied in :term:`pre-hook pod`)
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


.. _persistent-volume:

Persistent Volumes
------------------

In some cases it is necessary to add custom, persistent volumes.

In particular, these directories may be needed:

========================= ==============================================================================================
``/app/var/cms``           Contains CMS resources, in particular CSS and logos, used by our CMS. Most content has
                           been moved to git but some older installations still read resources from that location and
                           serve it via HTTP at ``/cms/*``.
``/app/var/lms``           The LMS module writes it's Scorm data to this directory. This directory must be made
                           persistent for all installation which have that module installed.
========================= ==============================================================================================


.. _persistent-volume-creation:

Creating a Persistent Volume
````````````````````````````

This creates a :term:`PVC` of size 1 GiB called ``cms`` which is mounted in the ``nice`` container at ``/app/var/cms``.

.. code::

    oc set volume dc/nice -c nice --add --name=cms --claim-name=cms --claim-size=1G --mount-path=/app/var/cms

You can list the PVCs using ``oc get pvc`` and you'll see the mounted volumes in the deployment config
using ``oc describe dc ${POD}``, section *Mount*.


Populating a Persistent Volume
``````````````````````````````

Here is how you copy the directory ``cms`` on your machine into a volume located at ``/var/app/cms``.


#. Find a running pod (a nice pod in this example)

   .. parsed-literal::

        # find a running pod (a nice pod in this case)
        $ oc get pods -l run=nice
        NAME             READY     STATUS    RESTARTS   AGE
        **nice-169-v2vsx**   2/2       Running   0          11m

#. Now, copy the content into the volume within that pod

   .. parsed-literal::

        oc cp -c nice cms **nice-169-v2vsx**:/app/var/cms


.. _persistent-volume-removal:

Removing a Persistent Volume
````````````````````````````

First remove the volume from the container. Then, remove the actual :term:`PVC`.

.. code::

        oc set volume dc/nice -c nice --remove --name=cms
        oc delete pvc cms
