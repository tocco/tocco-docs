Glossary
========

.. glossary::

    application property
        Application properties can be used to configure Nice. They are simple key, value pairs and look like this:

        .. code::

            my.property=configuration value

        The chapter :ref:`java-and-nice-params` describes how to set parameters in OpenShift. Alternatively, a property
        can be changed for a customer by adjusting the ``application.properties`` file. See next paragraph.

        While developing, you can edit the application.properties in the Nice Git repository located at
        ``customer/${CUSTOMER}/etc/application.properties``. If you don't want to commit the changes you can create
        an ``application.local.properties`` in the same directory. Settings made the the *local* properties file
        override the settings in the regular properties file.

    continuous delivery
    CD
        Continuous delivery is used to deploy our Nice installations.

        Our CD is powered by TeamCity and can be found at https://tc.tocco.ch.

    container
        A :term:`docker image` running in a :term:`pod`.

        Configuration is part of the :term:`deployment config`.

    deployment config
    DC
        The deployment config describes the containers associated with it. This includes image sources, resource limits,
        open ports, roll out strategy, triggers, etc.

         Accessible via ``oc (get|describe|edit|…) dc …``.

    docker image
        An image that contains an application and all run-time dependencies except the OS.

    exposed port
        Port that is made available to other pods or services.

        This is configured in the :term:`deployment config`.

    image stream
    IS
        Describes a docker repository. Pushing a docker image to it can be used to trigger an automatic deployment.

        Accessible via ``oc (get|describe|edit|…) is …``.

    image stream tag
        Describes a docker image tag. Defaults to ``latest``.

        Accessible via ``oc (get|describe|edit|…) imagestreamtag …``.

    Nginx
       `Nginx`_ is the web server used for as reverse proxy in front of Nice.

        Nginx is running in the same :term:`pod` as Nice.

        .. _Nginx: https://nginx.org/en/

    persistent volume claim
    PVC
        A persistent volume that can be mounted into one or more containers.

        Accessible via ``oc (get|describe|edit|…) pvc …``.

    pod
    PO
        A pod is one instance of the containers described in its :term:`deployment config`.

        Accessible via ``oc (get|describe|edit|…) pod …``.

    service
    SVC
        Used to make a service available in the network. It provides a DNS name for a service in a way that hides the
        fact that the service may be provided by several pods (multiple replicas).

        Accessible via ``oc (get|describe|edit|…) svc …``.

    Solr
        Solr is a search engine, Nice uses it to provide full-text search.

        Every Nice installation runs exactly one Solr :term:`pod`.

    route
        Provides a route to a service. This is used to make a service reachable via internet.

        Accessible via ``oc (get|describe|edit|…) route …``.
