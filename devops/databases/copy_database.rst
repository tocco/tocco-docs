Copy/Dump/Restore Database
==========================


Dump Database
-------------

.. code:: bash

    pg_dump -U postgres -h ${DB_SERVER} -Fc -f ~/_to_delete/nice2_${CUSTOMER}_$(date +"%Y_%m_%d").psql ${DATABASE};


Restore Database
----------------

.. code:: bash

    PGOPTIONS="-c synchronous_commit=off" pg_restore -j 4 -U postgres -h ${DB_SERVER} --role ${DB_USER} --no-owner --no-acl -d ${DB_NAME} ${DUMP_FILE_PATH}


Copy database using WITH TEMPLATE
---------------------------------

This is the fastest way to copy a database. Alternatively, you can dump and then restore the database.

.. warning::

    This requires that no one is connected to the database. Consequently, it isn't possible to copy a database of
    a running system.

Example
^^^^^^^

This example assumes that the customer name is *tocco* and DB name *nice2_tocco*.

#. switch to the right project

    .. code:: bash

        oc project toco-nice-${INSTALLATION}

#. check how many instance are running

    .. code:: bash

        oc get dc/nice -o go-template='{{.spec.replicas}}{{"\n"}}'

#. stop instance (if required)

    .. code:: bash

        oc scale --replicas=0 dc/nice

#. copy database

    .. code:: sql

        CREATE DATABASE ${NAME_OF_DB_COPY} WITH TEMPLATE ${SOURCE_DB_NAME};

    .. hint::

        If you get "source database '…' is being accessed by other users", try :ref:`killing the connections to the
        database <force-close-db-connection>` first.

    .. note::

        By convention, databases not used by a test or production systems should follow this naming pattern:
        ``nice_${CUSTOMER}_${YOUR_SHORT_NAME}_${YEAR}${MONTH}${DAY}``

5. restart instances (if previously stopped)

    .. code:: bash

        oc scale --replicas=${N} dc/nice

    Start ``${N}`` instances.
