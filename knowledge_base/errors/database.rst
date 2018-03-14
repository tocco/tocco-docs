Database
========

Can't Change Permissions on Extension
-------------------------------------

Error
^^^^^

.. code::

    … could not execute query: ERROR:  must be owner of extension lo
    Command was: COMMENT ON EXTENSION …

Cause
^^^^^

``pg_restore`` skips creation of the object itself, in this case **lo**, because it already exists. However, it still
tries to change the comment but doesn't have the permissions.

Solution 1 (short-term)
^^^^^^^^^^^^^^^^^^^^^^^

As long as only **COMMENT EXTENSION …** statements fail, this can be ignored safely.

Solution 2 (long-term)
^^^^^^^^^^^^^^^^^^^^^^

Remove the comments from the affected extensions to ensure future dumps won't contain **COMMENT ON …** statements.

Usually these two are affected:

.. code:: sql

    COMMENT ON EXTENSION lo IS NULL;
    COMMENT ON EXTENSION plpgsql IS NULL;

.. hint::

    Remove the comment in database **template1** to ensure new databases don't contain it. (A ``CREATE DATABASE xy``
    copies that DB.)

Full Error
^^^^^^^^^^

.. parsed-literal::

    $ **pg_restore -U nice_tocco -d nice_tocco nice_tocco.psql**
    pg_restore: connecting to database for restore
    pg_restore: creating SCHEMA public
    pg_restore: creating COMMENT SCHEMA public
    pg_restore: creating EXTENSION plpgsql
    pg_restore: creating COMMENT EXTENSION plpgsql
    pg_restore: [archiver (db)] Error while PROCESSING TOC:
    pg_restore: [archiver (db)] Error from TOC entry 2091; 0 0 COMMENT EXTENSION plpgsql
    pg_restore: [archiver (db)] could not execute query: **ERROR:  must be owner of extension plpgsql**
        **Command was: COMMENT ON EXTENSION** plpgsql IS 'PL/pgSQL procedural language';


Out of Shared Memory
--------------------

Error
^^^^^

.. parsed-literal::

    2017-12-19 03:38:20.691 ERROR nice2.tasks.MainTaskQueue [main-worker-2]
    Exception thrown during task run
    ch.tocco.nice2.persist.PersistException: SQL error: ERROR: **out of shared memory**
     Hint: You might need to increase max_locks_per_transaction.
           …

This error can also appear during ``pg_restore``\s.


Cause
^^^^^

There is a `fixed, global limit of locks <https://www.postgresql.org/docs/9.1/static/runtime-config-locks.html#UC-MAX-LOCKS-PER-TRANSACTION>`_
in Postgres. This error simply means that there are no more locks available. Usually, this is caused by poor code that
acquires hundreds of thousands of locks in a single transaction.

Solution
^^^^^^^^

Figure out what statement is  holding all these locks:

.. parsed-literal::

    SELECT l.\ **pid**, min(datname) AS db, count(*)
    FROM pg_locks AS l LEFT OUTER JOIN pg_stat_activity AS a ON l.pid = a.pid
    GROUP BY l.pid
    ORDER BY 3;

If you see a large count (>10,000) you may want to have a look at the query that's being executed.

.. parsed-literal::

    SELECT now() - query_start AS time, query
    FROM pg_stat_activity
    WHERE pid = **${pid}**

.. hint::

    **pid** is the number from the pid column of the first query.

Once you figured out what's causing the issue, so it can be fixed later, you may want to terminate the query:

.. parsed-literal::

    SELECT pg_terminate_backend(**${pid}**);


Postgres Connection Timeout
---------------------------

Error
^^^^^

.. code::

    HikariPool-1 - Connection is not available, request timed out after 30001ms.

often seen in connection with:

.. code::

    PersistenceException: org.hibernate.exception.JDBCConnectionException: Unable to acquire JDBC Connection


Cause
^^^^^

HikariCP, the connection pool used by Nice, has a fixed number of connections available. The error tells us that no
connection could be obtained from the pool because all of them are in use.


Analysis
^^^^^^^^

First, you need to figure out why there aren't enough connections around. For that you best collect some data:

.. hint::

    For the following SQL statements, disable paging and enable extended output to get output that's easier to read and
    copy paste:

    .. code::

        \pset pager off
        \x on

#. Check for custom HikariCP configurations:

           .. parsed-literal::

               $ oc set env dc/nice --list -c nice | grep '^NICE2_HIKARI_'
               NICE2_HIKARI_dataSource__databaseName=nice_toccotest
               NICE2_HIKARI_dataSource__password=************
               NICE2_HIKARI_dataSource__serverName=db1.tocco.cust.vshn.net
               NICE2_HIKARI_dataSource__user=nice_toccotest
               **NICE2_HIKARI_maximumPoolSize**\ =12
               NICE2_HIKARI_leakDetectionThreshold=30000

           **NICE2_HIKARI_maximumPoolSize** is what you should care about most. It tells you how large the pool is
           allowed to grow (=max. number of connections). If **NICE2_HIKARI_maximumPoolSize** doesn't appear in the
           output, the default value configured in Nice is used (currently 6). Take a look at `HikariCP's github page`_
           for information about available properties.

#. Take a look at how the connections are used

            .. code-block:: SQL

                SELECT
                    CASE WHEN state <> 'idle' THEN (now() - xact_start)::text ELSE 'idle' END AS "xact age",
                    CASE WHEN state <> 'idle' THEN (now() - query_start)::text ELSE 'idle' END AS "query age",
                    *
                FROM
                    pg_stat_activity
                WHERE
                    datname = current_database() AND pid <> pg_backend_pid()
                ORDER BY
                    greatest(query_start, xact_start) DESC;

            Connections with state **active** and **idle in transaction** are the ones you should care about. Also, look
            for unreasonably long-running queries and transactions (column **query age** and **xact age**).

#. Check for deadlocks

           If you suspect a deadlock, get the current state of the locks in addition.

           The following SQL statement have been obtained from Postgres' `Lock Monitoring`_ wiki page and slightly
           adjusted.

           .. code-block:: SQL

                SELECT blocked_locks.pid     AS blocked_pid,
                       blocked_activity.usename  AS blocked_user,
                       blocking_locks.pid     AS blocking_pid,
                       blocking_activity.usename AS blocking_user,
                       blocked_activity.query    AS blocked_statement,
                       blocking_activity.query   AS current_statement_in_blocking_process
                 FROM  pg_catalog.pg_locks         blocked_locks
                  JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
                  JOIN pg_catalog.pg_locks         blocking_locks
                      ON blocking_locks.locktype = blocked_locks.locktype
                      AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
                      AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
                      AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
                      AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
                      AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
                      AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
                      AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
                      AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
                      AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
                      AND blocking_locks.pid != blocked_locks.pid
                  JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
                 WHERE NOT blocked_locks.GRANTED;

           .. code-block:: SQL

                SET application_name='%your_logical_name%';
                SELECT blocked_locks.pid     AS blocked_pid,
                         blocked_activity.usename  AS blocked_user,
                         blocking_locks.pid     AS blocking_pid,
                         blocking_activity.usename AS blocking_user,
                         blocked_activity.query    AS blocked_statement,
                         blocking_activity.query   AS current_statement_in_blocking_process,
                         blocked_activity.application_name AS blocked_application,
                         blocking_activity.application_name AS blocking_application
                  FROM  pg_catalog.pg_locks         blocked_locks
                    JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
                    JOIN pg_catalog.pg_locks         blocking_locks
                        ON blocking_locks.locktype = blocked_locks.locktype
                        AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
                        AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
                        AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
                        AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
                        AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
                        AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
                        AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
                        AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
                        AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
                        AND blocking_locks.pid != blocked_locks.pid
                    JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
                  WHERE NOT blocked_locks.GRANTED;

           .. code-block:: SQL

                SELECT a.datname,
                         c.relname,
                         l.transactionid,
                         l.mode,
                         l.GRANTED,
                         a.usename,
                         a.query,
                         a.query_start,
                         age(now(), a.query_start) AS "age",
                         a.pid
                    FROM  pg_stat_activity a
                     JOIN pg_locks         l ON l.pid = a.pid
                     JOIN pg_class         c ON c.oid = l.relation
                    WHERE a.datname = current_database() AND a.pid <> pg_backend_pid()
                    ORDER BY a.query_start;

#. Find out where in Java the connections are used

            These are the options available:

            a) Create a thread dump (See :ref:`create-thread-dump`)
            b) Terminate connections to get a stack trace in Java (See :ref:`force-close-db-connection`)
            c) Enable leak detection:

               .. code::

                    oc set env -c nice dc/nice NICE2_HIKARI_leakDetectionThreshold=30000

               This tells HikariCP to log connections that have been taken out of the pool for more than 30,000 ms.
               HikariCP also logs a stack trace showing where the connection was obtained.

               .. warning::

                    Changing leakDetectionThreshold automatically restarts Nice.


Possible Measurements
^^^^^^^^^^^^^^^^^^^^^

a) Increase connection pool

       .. parsed-literal::

           oc set env -c nice dc/nice NICE2_HIKARI_maximumPoolSize=\ **15**

       This increases the connection pool to **15** connections.

       .. warning::

           Changing maximumPoolSize automatically restarts Nice. Also, do not increase the limit unnecessarily, `a
           higher pool size can decreases performance`_.

b) Split transactions into multiple, small transactions


.. _Lock Monitoring: https://wiki.postgresql.org/wiki/Lock_Monitoring
.. _HikariCP's github page: https://github.com/brettwooldridge/HikariCP
.. _a higher pool size can decreases performance: https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing
