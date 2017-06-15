Glossary
========

.. glossary::

    continuous delivery
    CD
        Continuous delivery is used to deploy our Nice2 installations.

        Our CD is powerd by TeamCity and can be found at https://tc.tocco.ch.

    container
        A :term:`docker image` running in a :term:`pod`.

        Configuration is part of the :term:`deployment config`.

    deployment config
    DC
        The deployment config describes the containers associated with it. This includes image sources, resource limits,
        open ports, roll out strategy, triggers, etc.

         Accessible via ``oc (get|describle|edit|…) dc …``.

    docker image
        An image that contains an application and all run-time dependencies except the OS.

    exposed port
        Port that is made available to other pods or services.

        This is configured in the :term:`deployment config`.

    Fluentd
        `Fluentd`_ is used for as a tool for central log collection on our OpenShift platform.

        .. _Fluentd: https://www.fluentd.org/

    image stream
    IS
        Describes a docker repository. Pushing a docker image to it can be used to trigger an automatic deployment.

        Accessible via ``oc (get|describle|edit|…) is …``.

    image stream tag
        Describes a docker image tag. Defaults to ``latest``.

        Accessible via ``oc (get|describle|edit|…) imagestreamtag …``.

    persistent volume claim
    PVC
        A persistent volume that can be mounted into one or more containers.

        Accessible via ``oc (get|describle|edit|…) pvc …``.

    pod
    PO
        A pod is one instance of the containers described in its :term:`deployment config`.

        Accessible via ``oc (get|describle|edit|…) pod …``.

    service
    SVC
        Used to make a service available in the network. It provides a DNS name for a service in a way that hides the
        fact that the service may be provided by several pods (multiple replicas).

        Accessible via ``oc (get|describle|edit|…) svc …``.

    route
        Provides a route to a service. This is used to make a service reachable via internet.

        Accessible via ``oc (get|describle|edit|…) route …``.
