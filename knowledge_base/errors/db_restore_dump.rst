Database Restore / Dump
=======================

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
