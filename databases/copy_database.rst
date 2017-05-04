Copy Database
=============

Dump and Restore
----------------

Dump the database and restore it again. Slower but can be done while the DB is in use.

Take a look at the :doc:`dump_database` and :doc:`restore_database` sections.

Using WITH TEMPLATE
-------------------

This is the fastest way to copy a database.

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

      CREATE DATABASE ${COPIED_DB_NAME} WITH TEMPLATE ${DATABASE_NAME};

   .. note:: By convention, databases not used by a test or production system should follow this naming pattern:
              **nice2_${CUSTOMER}_${YOUR_SHORT_NAME}_${YEAR}${MONTH}${DAY}**

4. restart instance (if previously stopped)

   .. code:: bash

      mgrsh start nice2-${DATABASE_NAME}
