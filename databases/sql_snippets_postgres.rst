SQL Snippets (Postgres)
=======================

Show Queries Running in Postgres
--------------------------------

show all queries:

.. code:: sql

    SELECT
      datname,
      now() - query_start as time,
      query 
    FROM
      pg_stat_activity
    WHERE 
      state = 'active';

show queries running on DB ``DB_NAME``:

.. code:: sql

    SELECT
      now() - query_start as time,
      query 
    FROM
      pg_stat_activity
    WHERE 
      datname = 'DB_NAME' AND state = 'active';


Forcibly Close Connections to DB
--------------------------------

Close all connections to DB ``DB_NAME``.

.. code:: sql

    SELECT
      pg_terminate_backend(pg_stat_activity.pid)
    FROM
      pg_stat_activity
    WHERE
      pg_stat_activity.datname = 'DB_NAME'
      AND pid <> pg_backend_pid(); 
