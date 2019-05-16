.. _scripts:

==================
Scripts (Postgres)
==================



n2sql-on-all-dbs
^^^^^^^^^^^^^^^^

Name of Scripts : n2sql-on-all-dbs
Location: installed on all DB servers
Description :


**Command: n2sql-on-all-dbs**

.. code::

    n2sql-on-all-dbs [-h] [--sql-file SQL_FILE] [--database REGEX]
                        [--postgres]
                        [--let-me-destroy-all-databases-at-once-by-enabling-read-write-mode]
                        [--autocommit] [--if-exists IF_EXISTS]
                        [--infinite | --repeat N | --repeat-until-no-row-touched]
                        [--no-skip-erred]
                        [--csv | --json | --simple | --time-only] [--timer]
                        [--dbuser USER] [--host HOST] [--port PORT]
                        [--no-password] [--threads N]
                        [sql]



n2change-db-owner
^^^^^^^^^^^^^^^^^

Name of Scripts : n2change-db-owner
Location: installed on all DB servers
Description : Change owner of DATABASE including all tables, sequences and large objects.

**Command: n2change-db-owner**
.. code::

    n2change-db-owner HOSTNAME DATABASE NEWUSER



n2passwd
^^^^^^^^

Name of Scripts : n2passwd
Location: installed on all DB servers
Description :

**Command: n2passwd**

.. code::

    Postgres Connection Options:
    -U, --dbuser=DBUSER
        Database user (e.g. 'postgres')

    -h, --host=DBHOST
        Database host (e.g. 'postgres.tocco.ch')

    -p, --port=DBPORT
        Database port (defaults to 5432)

    -w, --no-password
        Do not ask for a database password.

    Nice2 Options
    -u, --user=NICE2USER
        Nice2 user whose password is changed (defaults to 'tocco').

    -a, --all
        Change password on all databases on the database host, not just the ones given as DATABASE

    -b, --unblock-only
        Only unblock account, do not change password

    Debug Options:
    --no-commit
        Do not commit changes to the database, rollback instead.

    --show-sql
        Show data manipulation statements as they are executed.

    Examples:
     Change password for user 'developer' on database 'nice_master' (also unblocks account):
          /usr/local/bin/n2passwd -U postgres -h postgres.tocco.ch -u developer nice_master

     Change password for user 'tocco' on all databases (also unblocks account):
          /usr/local/bin/n2passwd -U postgres -h postgres.tocco.ch --all

     Unblock account 'pege' on database 'nice_arge':
          /usr/local/bin/n2passwd -U postgres -h postgres.tocco.ch --unblock-only -u pege nice_arge




