Routes / Hostnames
==================

Show Routes
-----------

.. code::

    $ oc get routes
    NAME                        HOST/PORT                  PATH      SERVICES    PORT      TERMINATION
    tocco                       tocco.tocco.ch                       nice        80-tcp    edge/Redirect
    tocco-backoffice.tocco.ch   backoffice.tocco.ch                  nice        80-tcp    edge/Redirect
    tocco-tocco.ch              tocco.ch                             nice        80-tcp    edge/Redirect
    tocco-webmodul.ch           webmodul.ch                          nice        80-tcp    edge/Redirect
    tocco-www.tocco.ch          www.tocco.ch                         nice        80-tcp    edge/Redirect
    tocco-www.webmodul.ch       www.webmodul.ch                      nice        80-tcp    edge/Redirect


.. _add-route:

Add Route / Hostname
--------------------

#. A template is used to create a new routes. You have to get it from the `Ansible Git Repository`_.

#. ``cd`` to the directory where ``nice-route-template.yml`` resides.

#. Create route:

    .. code::

        oc process -f nice-route-template.yml HOSTNAME=www.tocco.ch | oc create -f -

    ``HOSTNAME`` is the FQDN you want to add.

#. Create DNS entry if needed, see :doc:`dns`.

#. Issue SSL Certificate as described in the next section.

.. _Ansible Git Repository: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=openshift/nice-route-template.yml


.. _ssl-certificates:

SSL Certificates
----------------

.. _ssl-cert-issuance:

Issuance
^^^^^^^^

SSL certificates are issued automatically for routes with an appropriate annotation.

Obtain the name of the route (**${ROUTE}**)::

    oc get route

Add the annotation:

.. parsed-literal::

    oc annotate route/**${ROUTE}** kubernetes.io/tls-acme=true


.. warning::

    The DNS entry must exist and point to the right endpoint **before** adding the annotation.


Troubleshooting
^^^^^^^^^^^^^^^

In most cases where issuing a certificate fails, the DNS entry isn't correct or it wasn't correct when issuance was
first attempted. If that's the case, the issuance of certificates is paused.

List all routes in a project with paused SSL issuance:


    Command:

        .. code-block:: bash

           oc get route -o json | jq '.items[]|if .spec.path//"/" == "/" then [.metadata.name, .spec.host, .metadata.annotations."kubernetes.io/tls-acme-paused"//"false" ] else empty end'

    Sample output:

        Format: ``[ route, hostname, paused ], …``

        .. code-block:: javascript

           [
             "nice",               // <-- ${ROUTE}
             "tocco.tocco.ch",
             "true"                // <-- paused
           ],
           [
             "nice-tocco.ch",      // <-- ${ROUTE}
             "tocco.ch",
             "true"                // <-- paused
           ],
           [
             "nice-www.tocco.ch", // <-- ${ROUTE}
             "www.tocco.ch",
             "false"              // <-- not paused
           ]

In case a route is paused, ensure :ref:`the DNS entry is correct <verify-dns-records>` and then remove the paused annotation to force a retry.

Remove paused annotation:

.. parsed-literal::

    oc annotate route **${ROUTE}** kubernetes.io/tls-acme-paused-

.. warning::

   Issuing a certificate can take several minutes.


Remove Route
------------

.. code:: bash

    oc delete route tocco-www.tocco.ch
