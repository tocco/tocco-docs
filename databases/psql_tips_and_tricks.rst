Psql Tips and Tricks
====================

Getting Help
------------

Show SQL Syntax
^^^^^^^^^^^^^^^

``\h`` by itself shows all available statements. Use ``\h SELECT`` or ``\h ALTER DATABASE`` to see the full syntax
description.

.. code::

   \h ALTER DATABASE
   Command:     ALTER DATABASE
   Description: change a database
   Syntax:
   ALTER DATABASE name [ [ WITH ] option [ ... ] ]

   where option can be:

       ALLOW_CONNECTIONS allowconn
       CONNECTION LIMIT connlimit
       IS_TEMPLATE istemplate

   ALTER DATABASE name RENAME TO new_name

   ALTER DATABASE name OWNER TO { new_owner | CURRENT_USER | SESSION_USER }

   ALTER DATABASE name SET TABLESPACE new_tablespace

   ALTER DATABASE name SET configuration_parameter { TO | = } { value | DEFAULT }
   ALTER DATABASE name SET configuration_parameter FROM CURRENT
   ALTER DATABASE name RESET configuration_parameter
   ALTER DATABASE name RESET ALL

Show ``psql`` commands
^^^^^^^^^^^^^^^^^^^^^^

.. code::

   \?
   Informational
     (options: S = show system objects, + = additional detail)
     \d[S+]                 list tables, views, and sequences
     \d[S+]  NAME           describe table, view, sequence, or index
     \da[S]  [PATTERN]      list aggregates
     \dA[+]  [PATTERN]      list access methods
     \db[+]  [PATTERN]      list tablespaces
     \dc[S+] [PATTERN]      list conversions
     \dC[+]  [PATTERN]      list casts

   ...


Enable Extended Output
----------------------

Regular Output (columns)
^^^^^^^^^^^^^^^^^^^^^^^^

.. code::

   SELECT * FROM nice_history_domain_entity LIMIT 2;
    _nice_version | _nice_update_user |   _nice_update_timestamp   | _nice_create_user |   _nice_create_timestamp   |                xmlContent                | fk_history_version |      entityModel      | entityKey |   pk   | operation | fk_business_unit |    expires
    ---------------+-------------------+----------------------------+-------------------+----------------------------+------------------------------------------+--------------------+-----------------------+-----------+--------+-----------+------------------+---------------
             2 |                   | 2017-01-26 03:08:46.686+01 |                   | 2017-01-25 20:14:00.526+01 | 7bc7a4cdd4ff9a138fe9eefe061498e9b67b3322 |              74837 | History_domain_entity | 13536     | 680909 | UPDATED   |                  | 1501010040526
             2 |                   | 2017-01-26 03:08:54.836+01 |                   | 2017-01-25 20:14:01.032+01 | 4e1805cf7486501098f4bf45e024b88817add85a |              74838 | History_domain_entity | 13595     | 680915 | UPDATED   |                  | 1501010041032
   (2 rows)

Extended Output (rows)
^^^^^^^^^^^^^^^^^^^^^^

.. code::

   \x auto
   SELECT * FROM nice_history_domain_entity LIMIT 2;
   -[ RECORD 1 ]----------+-----------------------------------------
   _nice_version          | 2
   _nice_update_user      |
   _nice_update_timestamp | 2017-01-26 03:08:46.686+01
   _nice_create_user      |
   _nice_create_timestamp | 2017-01-25 20:14:00.526+01
   xmlContent             | 7bc7a4cdd4ff9a138fe9eefe061498e9b67b3322
   fk_history_version     | 74837
   entityModel            | History_domain_entity
   entityKey              | 13536
   pk                     | 680909
   operation              | UPDATED
   fk_business_unit       |
   expires                | 1501010040526
   -[ RECORD 2 ]----------+-----------------------------------------
   _nice_version          | 2
   _nice_update_user      |
   _nice_update_timestamp | 2017-01-26 03:08:54.836+01
   _nice_create_user      |
   _nice_create_timestamp | 2017-01-25 20:14:01.032+01
   xmlContent             | 4e1805cf7486501098f4bf45e024b88817add85a
   fk_history_version     | 74838
   entityModel            | History_domain_entity
   entityKey              | 13595
   pk                     | 680915
   operation              | UPDATED
   fk_business_unit       |
   expires                | 1501010041032

Details
^^^^^^^

``\x auto``: enable extended output if screen is too small
``\x on``:   always use extended output
``\x off``:  disable extended output

To use extended output by default add ``\x auto`` to ``~/.psqlrc``.

Change Password
---------------

.. code::

   \password USERNAME
   Enter new password: <PASSWORD>
   Enter it again: <PASSWORD>
