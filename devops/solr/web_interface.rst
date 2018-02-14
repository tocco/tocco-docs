Solr Web Interface
==================

Connecting to the Web Interface
-------------------------------

#. Switch to the right project

    .. parsed-literal::

        $ oc project toco-nice-**${INSTALLATION}**

#. Find the Solr pod

    .. parsed-literal::

        $ oc get pods -l app=solr -o name
        pods/**solr-23-3r3m8**

#. Forward Solr's port

    Forwards the port 8983 in container to port 8080 on your local machine.

    .. parsed-literal::

        $ oc port-forward **solr-23-3r3m8** 8080:8983

#. Connect to it using any browser

    Connect to ``http://localhost:8080``

    .. figure:: resources/solr_dashboard.png

        Solr's web interface
