Routes / Hostnames
==================

Show Routes
-----------

.. code::

    $ oc get routes
    NAME                        HOST/PORT                  PATH      SERVICES    PORT      TERMINATION
    tocco                       tocco.tocco.ch                       tocco       80-tcp    edge/Redirect
    tocco-backoffice.tocco.ch   backoffice.tocco.ch                  tocco       80-tcp    edge/Redirect
    tocco-tocco.ch              tocco.ch                             tocco       80-tcp    edge/Redirect
    tocco-webmodul.ch           webmodul.ch                          tocco       80-tcp    edge/Redirect
    tocco-www.tocco.ch          www.tocco.ch                         tocco       80-tcp    edge/Redirect
    tocco-www.webmodul.ch       www.webmodul.ch                      tocco       80-tcp    edge/Redirect


.. _add-route:

Add Route / Hostname
--------------------

#. A template is used to create a new routes. You have to get it from the `Ansible Git Repository`_.

#. ``cd`` to the directory where ``nice-route-template.yml`` resides.

#. Create route:

    .. code::

        oc process -f nice-route-template.yml HOSTNAME=www.tocco.ch SERVICE=nice | oc create -f -

    ``HOSTNAME`` is the FQDN you want to add and ``SERVICE`` is nice.

#. Create DNS entry if needed, see :doc:`dns`.

#. Issue SSL Certificate as described in the next section.

.. _Ansible Git Repository: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=openshift/nice-route-template.yml


.. _issue-ssl-certificate:

Issue SSL Certificate
---------------------

.. hint::

    * The :ref:`route <add-route>` and :doc:`DNS entry <dns>` must exist before you can issue a certificate.
    * See `Let's Encrypt Integration`_ section in the Appuio Community Documentation for more details.

#. Go to ``https://letsencrypt.appuio.ch/${HOSTNAME}`` (e.g. https\://letsencrypt.appuio.ch/www.tocco.ch)

#. Enter your OpenShift credentials

#. Verify you see a success message

.. _Let's Encrypt Integration: https://appuio-community-documentation.readthedocs.io/en/latest/letsencrypt-integration.html


Remove Routes
-------------

.. code:: bash

    oc delete route tocco-www.tocco.ch
