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


.. _ssl-certificates:

SSL Certificates
----------------

.. _ssl-cert-issuance:

Issuance
^^^^^^^^

.. admonition:: Deprecated
   :class: deprecated

   This is deprecated for use with Tocco but may still be used to enable
   SSL for other applications.

   To setup SSL for Tocco see :ref:`ansible-add-route`.

SSL certificates are issued automatically for routes with an appropriate annotation.

Obtain the name of the route (**${ROUTE}**)::

    oc get route

Add the annotation:

.. parsed-literal::

    oc annotate route/**${ROUTE}** kubernetes.io/tls-acme=true


.. warning::

    The DNS entry must exist and point to the right endpoint **before** adding the annotation.


.. _acme-troubleshooting:

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


.. _common.yaml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
