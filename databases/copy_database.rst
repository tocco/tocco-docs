Copy/Dump/Restore Database
==========================


Dump Database
-------------

.. code:: bash

   pg_dump -U postgres -h ${DB_SERVER} -Fc -f ~/_to_delete/nice2_${CUSTOMER}_$(date +"%Y_%m_%d").psql ${DATABASE};


Restore Database
----------------

.. code:: bash

   pg_restore -j 4 -U postgres -h ${DB_SERVER} --role ${DB_USER} --no-owner --no-acl -d ${DB_NAME} ${DUMP_FILE_PATH}


Copy database using WITH TEMPLATE
---------------------------------

This is the fastest way to copy a database. Alternatively, you can dump and then restore the database.

.. warning:: This requires that no one is connected to the database. Consequently, it isn't possible to copy a database of
             a running system.

Example
^^^^^^^

This example assumes that the customer name is *tocco* and DB name *nice2_tocco*.

1. stop instance (if required)

   .. code:: bash

      mgrsh stop nice2-${CUSTOMER_NAME}

2. kill connections to database (in case someone is still connected to it)

   .. code:: sql

      SELECT pg_terminate_backend(pg_stat_activity.pid)
      FROM pg_stat_activity
      WHERE pg_stat_activity.datname = '${DATABASE_NAME}'
            AND pid <> pg_backend_pid();


3. copy database

   .. code:: sql

      CREATE DATABASE ${NAME_OF_DB_COPY} WITH TEMPLATE ${SOURCE_DB_NAME};

   .. note:: By convention, databases not used by a test or production systems should follow this naming pattern:
              **nice2_${CUSTOMER}_${YOUR_SHORT_NAME}_${YEAR}${MONTH}${DAY}**

4. restart instance (if previously stopped)

   .. code:: bash

      mgrsh start nice2-${DATABASE_NAME}
