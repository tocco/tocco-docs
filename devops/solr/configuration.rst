Solr Configuration
==================

.. caution::

   :large-and-bold:`Deprecation Warning`

   This document describes how to adjust Solr running on OpenShift. This setup is depricated and Solr
   is now running on a server managed :hierra-repo:`via Puppet <infrastructure/solr.yaml>`. Unless
   there is a solr pod running in a project, this document is moot.


.. _solr-memory:

Solr Memory
-----------

Adjusting Memory
^^^^^^^^^^^^^^^^

#. Find the current memory settings

    .. parsed-literal::

        $ oc get dc solr -o yaml
        …
        spec:
          …
          template:
            …
            spec:
              containers:
              - name: solr
                …
                resources:
                  **limits**:
                    cpu: "3"
                    **memory**: 2Gi
                  **requests**:
                    cpu: 100m
                    **memory**: 200Mi
                …

#. Adjust it as required

    Adjust **limits.memory** to increase/decrease the memory available to Solr. **memory.requests** is used by OpenShift
    to figure out how much resource a pod is going to need, it should be set to approx. the average memory used.

    Example:

    .. code-block:: bash

        oc set resources --limits=memory=512Mi --requests=memory=256Mi dc/solr

    Check out OpenShifts introduction to `Requests and Limits`_ for more details.


Java Heap Memory
^^^^^^^^^^^^^^^^

Java heap memory is automatically set. The minimum value (-Xms…) is hardcoded and the maximum is set based on
**limits.memory** shown above. By default, a hardcoded factor in the OpenShift entrypoint script is used to calculate
the max. memory. Alternatively, the env. variable **MEMORY_FACTOR** can be set (**limits.memory** * **MEMORY_FACTOR** =
**MAX_JAVA_HEAP**).


Custom Configurations
---------------------

All configuration parameters available in ``/opt/solr/bin/solr.in.sh`` within the Docker image can be overridden using
environment variables by prefixing ``SOLR_PARAM_``.

For example, you can change ``GC_TUNE`` by setting:

.. code-block:: bash

    oc set env dc/solr SOLR_PARAM_GC_TUNE="-XX:NewRatio=3"

Take a look at the `sample config`_ used in the tests to see available properties.

.. _sample config: https://github.com/tocco/openshift-solr/blob/master/tests/sample_config.conf
