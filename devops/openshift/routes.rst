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


SSL Certificates
----------------

SSL certificates are issued automatically for route with an appropriate annotation.

Adding the annotation:

.. parsed-literal::

    oc annotate route/**${ROUTE}** kubernetes.io/tls-acme=true

For Nice installations, the templates for creating new installations and new routes already set this annotation. No
manual intervention is needed.

Remove Routes
-------------

.. code:: bash

    oc delete route tocco-www.tocco.ch
