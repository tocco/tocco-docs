SQL Snippets (Postgres)
=======================

Show Queries Running in Postgres
--------------------------------

Show all Queries
^^^^^^^^^^^^^^^^

.. code:: sql

    SELECT
      datname,
      now() - query_start as time,
      query
    FROM
      pg_stat_activity
    WHERE
      state = 'active';

Show Queries Running on DB
^^^^^^^^^^^^^^^^^^^^^^^^^^

Show queries on ``DB_NAME``:

.. code:: sql

    SELECT
      now() - query_start as time,
      query
    FROM
      pg_stat_activity
    WHERE
      datname = 'DB_NAME' AND state = 'active';

Show Long Running Queries
^^^^^^^^^^^^^^^^^^^^^^^^^

Show Queries that have been Running for more than 5 Minutes:

.. code:: sql

   SELECT
     pid,
     now() - query_start as time,
     datname,
     query
   FROM
     pg_stat_activity
   WHERE
     state = 'active'
     AND now() - query_start > interval '5m'
  ORDER BY
     query_start;

.. hint::

   You can terminate any running query using the number shown in the ``pid`` column:

   ``SELECT pg_terminate_backend(pid);``


.. _force-close-db-connection:

Forcibly Close Connections to DB
--------------------------------

Close all connections to DB ``DB_NAME``.

.. caution::

    This kills all connections to the Database, including connections from Nice and pg_dump!

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

.. _n2sql-on-all-dbs: https://git.tocco.ch/gitweb?p=nice2.git;a=blob;f=src/bin/n2sql-on-all-dbs.py

There is a script called `n2sql-on-all-dbs`_ for executing a query on all databases at once.


It is installed on **app03**:

    * execute given SQL on all Nice databases

        .. code::

            tocco@app03:~$ n2sql-on-all-dbs 'SELECT count(*) from _nice_binary'

    * execute sql in file **query.sql** on all Nice databases``

        .. code::

            tocco@app03:~$ n2sql-on-all-dbs -f query.sql


You can limit on what database it is executed via ``-d REGEX``:

    * execute on all test systems (name ends with **test**) ``-d '.*test$'``
    * execute on all but test systems (name doesn't end with **test**) ``-d '.*(?<!test)$'``

.. hint::

    Use ``n2sql-on-all-dbs --help`` for more details.
