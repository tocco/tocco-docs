SQL Snippets (Postgres)
=======================

Show Queries Running in Postgres
--------------------------------

Show Queries Running on DB
^^^^^^^^^^^^^^^^^^^^^^^^^^

Show open transactions and queries running on current database:

.. code:: sql

    SELECT
      pid,
      now() - xact_start as "tx time",
      now() - query_start as "query time",
      state,
      query
    FROM
      pg_stat_activity
    WHERE
      state <> 'idle'
      AND datname = current_database()
      AND pid <> pg_backend_pid()
    ORDER BY
      xact_start;

Show Long Running Queries
^^^^^^^^^^^^^^^^^^^^^^^^^

Show queries associated with transactions that have been open for more than 5 minutes:

.. code:: sql

   SELECT
     pid,
     now() - xact_start as "tx time",
     now() - query_start as "query time",
     datname,
     state,
     query
   FROM
     pg_stat_activity
   WHERE
     state <> 'idle'
     AND now() - xact_start > interval '5m'
  ORDER BY
     xact_start;

.. hint::

   You can terminate any running query using the number shown in the ``pid`` column:

   ``SELECT pg_terminate_backend(pid);``


.. _force-close-db-connection:

Forcibly Close Connections to DB
--------------------------------

Close all connections to DB ``DB_NAME``.

.. caution::

    This kills all connections to the database, including connections from Nice and pg_dump!

.. code:: sql

    SELECT
      pg_terminate_backend(pg_stat_activity.pid)
    FROM
      pg_stat_activity
    WHERE
      pg_stat_activity.datname = 'DB_NAME'
      AND pid <> pg_backend_pid();


Execute SQL on Multiple Databases
---------------------------------

See the command description here for the scripts commands :ref:`commands-scripts`


There is a script called `n2sql-on-all-dbs`_ for executing a query on all databases at once.


It is installed and updated via Ansible **on all DB servers**:

    * execute given SQL on all Nice databases::

        n2sql-on-all-dbs 'SELECT count(*) from _nice_binary'



    * execute sql in file **query.sql** on all Nice databases``::

        n2sql-on-all-dbs -f query.sql


You can limit on what database it is executed via ``-d REGEX``:

    * execute on all test systems (name ends with **test**) ``-d '.*test$'``
    * execute on all but test systems (name doesn't end with **test**) ``-d '.*(?<!test)$'``

.. hint::

    Use ``n2sql-on-all-dbs --help`` for more details.

.. _n2sql-on-all-dbs: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=roles/postgres-client-utils/files/bin/n2sql-on-all-dbs
