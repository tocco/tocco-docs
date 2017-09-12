OpenShift Client (oc)
=====================

Setting up oc
-------------

1. install oc

   Download the client tools from the `OpenShift download page`_, extract it and move the ``oc`` binary into your ``$PATH``
   (e.g. into ``~/bin/``).

   .. _OpenShift download page: https://www.openshift.org/download.html

2. enable autocompletion

    Execute ``oc completion -h`` and follow the instructions.

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

Show Resources
--------------

* List all resources

  .. code:: bash

     $ oc get all
     NAME              DOCKER REPO                                    TAGS              UPDATED
     is/pege           172.30.1.1:5000/appuio-demo4441/pege      test,production   28 hours ago
     …

     NAME            REVISION   DESIRED   CURRENT   TRIGGERED BY
     dc/pege         7          0         0         config,image(pege:production)
     …

     NAME              DESIRED   CURRENT   READY     AGE
     rc/pege-1         0         0         0         13d
     …

     NAME                HOST/PORT           PATH      SERVICES          PORT       TERMINATION
     routes/pege         pege.tocco.ch                 pege         80-tcp     edge/Redirect
     …

     NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)                   AGE
     svc/solr               172.30.64.69     <none>        8983/TCP                  13d
     …

     NAME                      READY     STATUS      RESTARTS   AGE
     po/pegetest-14-edtob      2/2       Running     0          1d
     …

* List resources of a particular type

  .. code:: bash

     $ oc get pod
     NAME               READY     STATUS      RESTARTS   AGE
     nice-13-deploy     0/1       Error       0          1d
     nice-13-hook-pre   0/1       Completed   0          1d
     nice-14-edtob      2/2       Running     0          1d
     solr-10-qd9u8      1/1       Running     0          2d


* Show details for a resource

  #. show available deployment configs (or any other resource type)

     .. code::

        $ oc get dc
        NAME        REVISION   DESIRED   CURRENT   TRIGGERED BY
        nice        7          0         0         config,image(pege:production)
        solr        14         0         0         config,image(solr:stable)

  #. use the **NAME** column to retrieve more details

     .. code::

        $ oc describe dc pege
        Name:           pege
        Namespace:      appuio-demo4441
        Created:        13 days ago
        Labels:         run=pege
        Annotations:    <none>
        Latest Version: 7
        Selector:       run=pege
        Replicas:       0
        Triggers:       Config, Image(pege@production, auto=true)
        Strategy:       Recreate
        …


Edit resources
--------------

Take a look at :doc:`edit_resources`.
