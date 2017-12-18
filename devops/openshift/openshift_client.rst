OpenShift Client (oc)
=====================

Setting up oc
-------------

1. install oc

   Download the client tools from the `OpenShift download page`_, extract it and move the ``oc`` binary into your ``$PATH``
   (e.g. into ``~/bin/``).

   .. _OpenShift download page: https://www.openshift.org/download.html

2. enable autocompletion

    On most Linux based systems:

    .. code::

        oc completion bash | sudo bash -c 'cat >/etc/bash_completion.d/oc'
        exit # and open a new terminal

    For non-Linux systems use ``oc completion --help`` for further information.

3. login

   .. code:: bash

       oc login https://console.appuio.ch
       Username: <USERNAME>
       Password: <PASSWORD>

   .. todo:: update endpoint for private cloud

4. switch to the right project

    switch to the Nice project of ${INSTALLATION}:

    .. code::

        oc project toco-nice-${INSTALLATION}

    .. hint::

        You can list all available projects like this:

        .. code::

            oc get projects


Resource Types
--------------

To get a list of all resources use ``oc get``.

Here are the resource types you'll need:

======= =================================================================================================================
 Type   Description
======= =================================================================================================================
 dc      **D**\eployment **c**\onfiguration: A template for a pod. If you need to change a pod, change this
         configuration. Changes are automatically deployed.

 pod     A pod is an instance of a Deployment Configuration (AKA dc).

         * Start and stop them using ``oc scale replicas=N dc/nice`` (``N`` → number of instances)
         * Change the dc to change the configuration of a pod

 route   A DNS route

         .. code::

            oc get route
            NAME      HOST/PORT          PATH      SERVICES   PORT      TERMINATION     WILDCARD
            nice      abc.tocco.ch                 nice       80-tcp    edge/Redirect   None
            nice      www.abc.ch                   nice       80-tcp    edge/Redirect   None

 svc     Service (**svc**): Represents a set of pods that provide a service. Allows talking to Solr, for instance,
         without having to worry about in what pod it is running in or how many instances are running.

 is      **I**\mage **s**\tream: This is a docker image that has been pushed to OpenShift. In the world of docker, they
         are called repositories.
======= =================================================================================================================


.. _list-resources:

List Resources
--------------

Use ``oc get TYPE`` to get all list of the resource of a certain type or ``oc get all`` to show all of them.

Example
^^^^^^^

  .. code:: bash

    $ oc get pod
    NAME            READY     STATUS    RESTARTS   AGE
    nice-25-kchv3   2/2       Running   0          3h
    solr-2-gt5tg    1/1       Running   0          1h


Describe a Resource in Detail
-----------------------------

Use ``oc describe TYPE RESOURCE`` to show details about a specific pod/dc/is/….

.. hint:: :ref:`list-resources` shows how to obtain the RESOURCE name.

Example
^^^^^^^

     .. code::

        $ oc describe pod nice-25-kchv3
        Name:                   nice-25-kchv3
        Namespace:              toco-nice-test212
        Security Policy:        restricted
        Node:                   node19.prod.zrh.appuio.ch/172.17.176.161
        Start Time:             Wed, 18 Oct 2017 13:07:00 +0200
        Labels:                 deployment=nice-25
                                deploymentconfig=nice
                                run=nice
        …


Edit Resources
--------------

Use ``oc edit TYPE RESOURCE`` to edit a specific pod/dc/is/….

.. hint:: :ref:`list-resources` shows how to obtain the RESOURCE name.

Example
^^^^^^^

    #. Open config in editor: ``oc edit pod nice-25-kchv3``
    #. Make any changes you want to the configuration.
    #. Save changes and exit in order to trigger a deployment.

See document :doc:`edit_resources` for all the details.


Open Shell in Pod
-----------------

.. code::

    oc rsh -c nice PODNAME bash

``-c`` specifies the container name, use ``-c nginx`` to enter the nginx container or ``oc rsh PODNAME bash`` to enter
a Solr pod (has only one container).


Copy File from Pod
------------------
  
.. code::
  
    oc cp -c nice PODNAME:/path/to/file.txt ~/destination/folder/


Synchronize Folder with Pod
---------------------------

.. code::
  
    oc rsync -c nice PODNAME:/path/to/folder ~/destination/folder/


Manually Deploy
---------------

Deploy latest version of Nice:

.. code::

    oc rollout latest dc/nice


Retry Failed Deployment
-----------------------

Retry failed deployment of Nice:

.. code::

    oc deploy --retry dc/nice


Open a Remote Shell in a Pod
----------------------------

To get a shell within a Nice pod use ``oc rsh -c nice POD``.

Example
^^^^^^^

.. code::

    $ oc rsh -c nice nice-25-kchv3
    nice-25-kchv3:/app $ …

Open a shell in the Nginx container using ``oc rsh -c nginx nice-25-kchv3`` or in the Solr Pod using
``oc rsh solr-2-gt5tg``.

Access Log Files in Nice Pod
----------------------------

.. code::

    oc exec -c nice PODNAME -- tail -n +0 var/log/nice.log |less


Start PSQL
----------

This open the database the pod uses.

.. code::

    $ oc rsh -c nice PODNAME psql
    psql (9.4.13, server 9.5.9)

    nice_test212=> …
