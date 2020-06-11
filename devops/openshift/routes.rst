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

.. hint::

   Expect certificate issuance to take up to 15 minutes.

Check for missing TLS certificates in the OpenShift project::

    oc project toco-nice-${INSTALLATION}
    oc get route -o json | jq -r '
        .items[]
        |if .metadata.labels."acme.openshift.io/temporary" == "true" or (.spec.tls|has("key")) then
           empty
         else
             "name: " + .metadata.name + "\n" +
             "host: " + .spec.host + "\n" +
             "path: " + (.spec.path//"/") + "\n" +
             "kubernetes.io/tls-acme: " + .metadata.annotations."kubernetes.io/tls-acme" + "\n" +
             "acme.openshift.io/status: " + "\n" +
             "  " + (.metadata.annotations."acme.openshift.io/status"|split("\n")|join("\n  "))
        end'

Sample output:

.. code-block:: yaml

    name: nice-any.tocco.ch
    host: any.tocco.ch
    path: /
    kubernetes.io/tls-acme: true  # <-- Certificates is only issued if this is enabled. Set
                                  #     by Ansible.
    acme.openshift.io/status:
      provisioningStatus:
        earliestAttemptAt: "2020-06-10T06:20:17.55709836Z"
        orderStatus: pending      # <-- See below
        orderURI: https://acme-v02.api.letsencrypt.org/acme/order/87247129/3706292941
        startedAt: "2020-06-10T06:20:17.55709836Z"

Meaning of ``orderStatus``:

================ ================================================================
 ``pending``      Validation has not yet succeeded.

                  If the status is stuck in pending state, …

                  * … wait a bit longer. Issuance can take 15 minutes.

                  * … :ref:`verify that the DNS entry is correct
                    <verify-dns-records>`.

                  * … remove the route::

                          oc delete route ${ROUTE_NAME}

                    and recreate it::

                        ansible-playbook playbook.yml -t route -l ${INSTALLATION}

 ``ready``        Validation has succeeded.

                  Certificate should be issued within minutes.

 ``processing``   Certificate is being issued.

                  Certificate should be ready within minutes.

 ``valid``        Certificate is valid and has been added to the route.

                  TLS connections should work.

 ``…``            See https://godoc.org/golang.org/x/crypto/acme#pkg-constants
================ ================================================================


Related documentation:

* `Let's Encrypt Integration`_ in the Appuio documentation.
* `openshift-acme`_ , the ACME controller used on OpenShift.


.. _common.yaml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
.. _openshift-acme: https://github.com/tnozicka/openshift-acme#openshift-acme
.. _Let's Encrypt Integration: https://docs.appuio.ch/en/latest/letsencrypt-integration.html
