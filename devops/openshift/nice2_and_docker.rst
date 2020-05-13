Nice2 and Docker
================

Build Docker Image - the Manual Way
-----------------------------------

#. Change working directory

    .. parsed-literal::

        cd **${ROOT_DIR_OF_NICE2_REPOSITORY}**

#. Assembly build

    .. parsed-literal::

        rm nice2-customer-\*-1.0-SNAPSHOT.tar.gz
        mvn -pl customer/**${CUSTOMER}** -am clean install -T1.5C -DskipTests -P assembly
        mv ./customer/**${CUSTOMER}**/target/nice2-customer-**${CUSTOMER}**-1.0-SNAPSHOT.tar.gz .

#. Building the Docker image

    .. parsed-literal::

        docker build -t **nice** --build-arg NICE2_VERSION=$(<current-version.txt) --build-arg NICE2_REVISION=$(git rev-parse HEAD) .

    The resulting image is called **nice**.


Pull a Docker Image from VSHN's Registry
----------------------------------------

#. Login

   .. code-block:: bash

       oc whoami -t | docker login registry.appuio.ch -u any --password-stdin

#. Find Image

   .. parsed-literal::

      oc project **${PROJECT}**
      oc get is
      NAME      DOCKER REPO                          TAGS      UPDATED
      nice      172.30.1.1:5000\ :red:`/toco-nice-stn/nice`   :blue:`latest`    2 weeks ago

   .. hint::
   
      An ``oc describe …`` on :term:`RC`\ s and :term:`pod`\ s will also reveal the used images.
   
      :term:`Solr` and :term:`Nginx` images can be found in the project ``toco-shared-imagestreams``.

#. Pull image

   .. parsed-literal::

      docker pull registry.appuio.ch\ :red:`/toco-nice-stn/nice`::blue:`latest`



Extend an existing Image
---------------------------

#. Login

   .. code-block:: bash

       oc whoami -t | docker login registry.appuio.ch -u any --password-stdin

#. Create an empty directory

   .. code::

      mkdir image
      cd image

#. Create a ``Dockerfile``

   .. parsed-literal::
   
      FROM registry.appuio.ch\ :red:`/toco-nice-stn/nice`::blue:`latest`
   
      COPY entrypoint.py /usr/local/bin/entrypoint.py
      RUN chmod +x /usr/local/bin/entrypoint.py
   
   Replace ``COPY`` and ``RUN`` with instructions useful to you. See the `Dockerfile reference`_ for details.

#. Build Image

   .. parsed-literal::

      docker build -t **nice** .

   The resulting image is called **nice**.
      

Deploying a Docker Image
------------------------

#. Login

   .. code-block:: bash

       oc whoami -t | docker login registry.appuio.ch -u any --password-stdin

#. Tag image

   This additionally tags the image named **nice** with the name *registry.appuio.ch/…*.

   .. parsed-literal::

      docker tag **nice** registry.appuio.ch/toco-nice-\ **${INSTALLATION}**\ /nice

#. Deploy image

   .. parsed-literal::

      docker push registry.appuio.ch/toco-nice-\ **${INSTALLATION}**\ /nice

   Deployment is automatically started once the image is pushed.


Running the Image Locally
-------------------------

.. parsed-literal::

    docker run --rm -p 8080:8080 -e NICE2_HIKARI_dataSource.serverName=\ **${DB_SERVER}** \\
      -e NICE2_LOGBACK_CONFIG=\ **logback_terminal** -e NICE2_HIKARI_dataSource.databaseName=\ **${DB_NAME}** \\
      -e NICE2_HIKARI_dataSource.user=\ **${DB_USER}** -e NICE2_HIKARI_dataSource.password=\ **${DB_PASSWORD}** \\
      -e NICE2_JAVA_OPT\_-Dch.tocco.nice2.runenv=\ **development** -e NICE2_HIKARI_dataSource.sslMode=require \\
      **${DOCKER_IMAGE_NAME}**

.. hint::

   If you run Postgres in a Docker container called *pg*, as described below, use this to link the Nice container to it::

       -e NICE2_HIKARI_dataSource.serverName=pg -e NICE2_HIKARI_dataSource.user=nice \
       -e NICE2_HIKARI_dataSource.password=nice -e NICE2_HIKARI_dataSource.sslMode=disable --link pg

   Linking container ``pg`` (``--link pg``) makes the container available with the host name ``pg`` within the Nice container.

Important environment variables:

+-----------------------------+----------------------------------------------------------------------------------------+
| Name                        | Description                                                                            |
+=============================+========================================================================================+
| NICE2_LOGBACK_CONFIG        | The logback configuration to be used.                                                  |
|                             |                                                                                        |
|                             | Currently available:                                                                   |
|                             |                                                                                        |
|                             |   * ``logback_terminal``:                                                              |
|                             |       Write logs to stdout in human readable form (severity warning and above only)    |
|                             |                                                                                        |
|                             |   * ``logback_terminal_info``:                                                         |
|                             |       Identical to logback_terminal except that info message are logged too            |
|                             |                                                                                        |
|                             |   * ``logback_json``:                                                                  |
|                             |       Write logs in JSON format (default)                                              |
+-----------------------------+----------------------------------------------------------------------------------------+
| NICE2_HIKARI_dataSource\    | Host name of the Database server                                                       |
| .serverName                 |                                                                                        |
+-----------------------------+----------------------------------------------------------------------------------------+
| NICE2_HIKARI_dataSource\    | Database name                                                                          |
| .databaseName               |                                                                                        |
+-----------------------------+----------------------------------------------------------------------------------------+
| NICE2_HIKARI_dataSource\    | Database user                                                                          |
| .user                       |                                                                                        |
+-----------------------------+----------------------------------------------------------------------------------------+
| NICE2_HIKARI_dataSource\    | Database user password                                                                 |
| .password                   |                                                                                        |
+-----------------------------+----------------------------------------------------------------------------------------+
| NICE2_JAVA_OPT\_-Dch.tocco\ | Run environment (``development``, ``test``, ``production`` or ``update``)              |
| .nice2.runenv               |                                                                                        |
+-----------------------------+----------------------------------------------------------------------------------------+
| NICE2_APP_hiveapp\          | Timeout for startup checks. Set this to an impossibly high value to ensure application |
| .StarterExecutor\           | doesn't abort if a startup task doesn't complete.                                      |
| .starterTimeoutInMinutes    |                                                                                        |
+-----------------------------+----------------------------------------------------------------------------------------+
| NICE2_APP_nice2\            | Use this to overwrite the default business units.                                      |
| .dbrefactoring\             |                                                                                        |
| .businessunits              |                                                                                        |
+-----------------------------+----------------------------------------------------------------------------------------+


Create DB or Update Schema
--------------------------

In order to run DB refactoring, which will create or update a DB, run the same command as for starting the container
but execute the ``dbref`` command within the container.

.. parsed-literal::

    docker run --rm -p 8080:8080 -e NICE2_HIKARI_dataSource.serverName=${DB_SERVER} \\
      -e NICE2_LOGBACK_CONFIG=logback_terminal -e NICE2_HIKARI_dataSource.databaseName=${DB_NAME} \\
      -e NICE2_HIKARI_dataSource.user=${DB_USER} -e NICE2_HIKARI_dataSource.password=${DB_PASSWORD} \\
      -e NICE2_JAVA_OPT\_-Dch.tocco.nice2.runenv=development -e NICE2_HIKARI_dataSource.sslMode=require \\
      ${DOCKER_IMAGE_NAME} **dbref**


Postgres
--------

If you need Postgres on your machine, A simple solution is to run Postgres in Docker.

Available Postgres Images
^^^^^^^^^^^^^^^^^^^^^^^^^

Base images:

========== =================================================== ================================================
 Postgres   Git Repository and Branch                           Docker Image URL
========== =================================================== ================================================
 9.5        `nice2-postgres`_:9.5                               registry.gitlab.com/toccoag/nice2-postgres:9.5
 11         `nice2-postgres`_:11                                registry.gitlab.com/toccoag/nice2-postgres:11
            ... (See `available branches on nice2-postgres`_    ...
 ...        for a complete list)
========== =================================================== ================================================

Extended images for testing and testing only:

========== ======================= =============================================================== ==================================================================================
 Postgres   Included Database       Git Repository and Branch                                       Docker Image URL
========== ======================= =============================================================== ==================================================================================
 9.5        *none*                 `nice2-postgres-for-tests`_:postgres-9.5                         registry.gitlab.com/toccoag/nice2-postgres-for-tests:postgres-9.5
 9.5        demo.tocco.ch (v2.17)  `nice2-postgres-for-tests`_:postgres-9.5-nice-2.17-demo          registry.gitlab.com/toccoag/nice2-postgres-for-tests:postgres-9.5-nice-2.17-demo
 11         *none*                 `nice2-postgres-for-tests`_:postgres-11                          registry.gitlab.com/toccoag/nice2-postgres-for-tests:postgres-11
 11         demo.tocco.ch (v2.17)  `nice2-postgres-for-tests`_:postgres-11-nice-2.17-demo           registry.gitlab.com/toccoag/nice2-postgres-for-tests:postgres-11-nice-2.17-demo
 ...        ...                    ... (See `available branches on nice2-postgres-for-tests`_
                                   for a complete list)
========== ======================= =============================================================== ==================================================================================

.. important::

   The ``nice2-postgres-for-tests`` images are far faster but the have certain safety features like ``fsync`` disabled.
   Use them only if complete data loss is acceptable.


.. _nice2-postgres-for-tests: https://gitlab.com/toccoag/nice2-postgres-for-tests
.. _nice2-postgres: https://gitlab.com/toccoag/nice2-postgres
.. _available branches on nice2-postgres-for-tests: https://gitlab.com/toccoag/nice2-postgres-for-tests/branches
.. _available branches on nice2-postgres: https://gitlab.com/toccoag/nice2-postgres/branches

Start Postgres
^^^^^^^^^^^^^^

You can start Postgres locally using Docker like this:

.. code-block:: bash

    mkdir dumps/
    docker run --rm --name pg -d -p 5432:5432 -v "$PWD/data:/data" -v "$PWD/dumps:/custom_dumps" registry.gitlab.com/toccoag/nice2-postgres:9.5

This uses the directory ``data/`` in the current directory to store the database. Skip it if you have no need for the data to be persistent.
Additionally, ``custom_dumps/`` in the current directory is made available as ``/custom_dumps/`` in the container.


SQL Console
^^^^^^^^^^^

Once postgres is running you can open an SQL console (``psql``):

.. parsed-literal::

    docker exec -it pg **psql**

Here you can create DB for instance:

.. code:: sql

   CREATE DATABASE test WITH OWNER nice;

Or you can restore a dump that you copied into the ``dumps/`` directory::

    $ ls -lh custom_dumps/
    total 20M
    -rw-r--r-- 1 user user 20M Jun 29 15:12 dump.psql
    docker exec -it -w /custom_dumps pg pg_restore --role --no-owner --no-acl nice -d DB dump.psql

.. hint::

   The Docker image creates an empty DB called ``nice`` owned by user ``nice`` whose password is also ``nice``.


Connect From Nice2
^^^^^^^^^^^^^^^^^^

You connect to Postgres on host ``localhost`` port ``5432``:

``hikaricp.local.properties``:

.. parsed-literal::

   dataSource.serverName=localhost
   dataSource.databaseName=\ **${DB_NAME}**
   dataSource.password=nice
   dataSource.user=nice

.. hint::

   For images with test data, the password of all users, including user *tocco*, is **nice**.
