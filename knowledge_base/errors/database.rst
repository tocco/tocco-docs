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
