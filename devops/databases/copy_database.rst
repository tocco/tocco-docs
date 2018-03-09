Copy/Dump/Restore Database
==========================

.. hint::

        If you want restore a backup have a look at :doc:`../backups/database`.

Dump Database
-------------

.. code-block:: bash

    pg_dump -U postgres -h ${DB_SERVER} -Fc -f ~/_to_delete/nice2_${CUSTOMER}_$(date +"%Y_%m_%d").psql ${DATABASE};


.. _restore-database:

Restore Database
----------------

**\*.psql** files:

    .. code-block:: bash

        PGOPTIONS="-c synchronous_commit=off" pg_restore -j 4 -U postgres -h ${DB_SERVER} --role ${DB_USER} --no-owner --no-acl -d ${DB_NAME} ${DUMP_FILE_PATH}

**\*.dump.gz** files:

    .. code-block:: bash

        gzip -cd ${DUMP_FILE_PATH} | PGOPTIONS="-c synchronous_commit=off" pg_restore -j 4 -U postgres -h ${DB_SERVER} --role ${DB_USER} --no-owner --no-acl -d ${DB_NAME}

.. hint::

    For the restore commands above, You'll need the password for *postgres*  The current password can be found in the
    `tocco_hieradata repository`_ (key: ``profile_postgresql::server::password``. If you're lacking access, ask
    someone with the appropriate permissions for it.


Copy database using WITH TEMPLATE
---------------------------------

This is the fastest way to copy a database. Alternatively, you can dump and then restore the database.

.. parsed-literal::

    CREATE DATABASE **${TARGET_DB}** WITH TEMPLATE **${SOURCE_DB}**;

.. warning::

    This requires that no one is connected to the database. Consequently, it isn't possible to copy a database of
    a running system.

Example
^^^^^^^

This example assumes that the customer name is *tocco* and DB name *nice2_tocco*.

#. switch to the right project

    .. code-block:: bash

        oc project toco-nice-${INSTALLATION}

#. check how many instance are running

    .. code-block:: bash

        oc get dc/nice -o go-template='{{.spec.replicas}}{{"\n"}}'

#. stop instance (if required)

    .. code-block:: bash

        oc scale --replicas=0 dc/nice

#. copy database

    .. parsed-literal:: sql

        CREATE DATABASE **${NAME_OF_DB_COPY}** WITH TEMPLATE **${SOURCE_DB_NAME}**;

    .. hint::

        If you get "source database '…' is being accessed by other users", try :ref:`killing the connections to the
        database <force-close-db-connection>` first.

    .. note::

        By convention, databases not used by a test or production systems should follow this naming pattern:
        ``nice_${CUSTOMER}_${YOUR_SHORT_NAME}_${YEAR}${MONTH}${DAY}``

5. restart instances (if previously stopped)

    .. parsed-literal:: bash

        oc scale --replicas=\ **${N}** dc/nice

    Start **${N}** instances.


.. _tocco_hieradata repository: https://git.vshn.net/tocco/tocco_hieradata/blob/master/database.yaml
