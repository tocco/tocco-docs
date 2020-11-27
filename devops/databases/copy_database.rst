Copy/Dump/Restore Database
==========================

.. hint::

        If you want restore a backup have a look at :doc:`../backups/database`.

.. _ansible-copy-db:

Copy Database via Ansible
-------------------------

.. hint::

    * If you haven't yet, :ref:`setup-ansible` first.
    * If you want to restore to localhost (the default), also :ref:`setup-postgres`.

Usage::

    cd ${ANSIBLE_REPO}/tocco
    ansible-playbook playbooks/copy_db.yml -l INSTALLATION[,INSTALLATION]... [-e target_server=TARGET] [-e no_binaries=yes]

===================== ==========================================================
 **INSTALLATION**      | Name of the installation, for instance *agogis*
                       | or *agogistest*.
 **TARGET**:           | Host on which to restore the DB, for instance
                       | *postgres.tocco.ch*. If omitted, restore is done
                       | on localhost via Unix socket.
 **no_binaries=yes**   | Omit mail archive and binaries from copy.
                       |
                       | **Errors during restore are expected as referenced**
                       | **rows will be missing.**
===================== ==========================================================


Example:

.. parsed-literal::

    $ cd ${ANSIBLE_REPO}/tocco
    $ ansible-playbook playbooks/copy_db.yml -l magentatest -e target_server=postgres.tocco.ch
    PLAY [tocco_installations]

    *... omitted ...*

    TASK [fetch DB size]
    ok: [magentatest -> db1.tocco.cust.vshn.net]

    TASK [print DB details]
    ok: [magentatest] =>
      msg: \|-
        source:
          db: nice_magentatest
          size: 379 MB
        target:
          server: postgres.tocco.ch
          db: nice_magentatest_20200422t145625_pgerber
          user: nice_magentatest
          password: XXXXXXXXXXXXXXXXXXXXXXXX

        ---

        **dataSource.serverName=postgres.tocco.ch**
        **dataSource.databaseName=nice_magentatest_20200422t145625_pgerber**
        **dataSource.user=nice_magentatest**
        **dataSource.password=XXXXXXXXXXXXXXXXXXXXXXXX**

    *... omitted ...*

    TASK [copy_database]
    changed: [magentatest]

    TASK [details]
    ok: [magentatest] =>
      msg: \|-
        **duration: 0:05:20.979651**

    PLAY RECAP
    magentatest                : ok=10   changed=3    unreachable=0    failed=0

    Playbook run took 0 days, 0 hours, 5 minutes, 58 seconds

Use the DB configuration printed **boldly** by copying it to *hikaricp.local.properties*.

Passwords are generated deterministically and will always be the same for the same
installation and server. For instance, any database of *agogistesto on *localhost*
will always have the same password.


Dump Database
-------------

.. code-block:: bash

    pg_dump -U postgres -h ${DB_SERVER} -Fc -f ~/_to_delete/nice2_${CUSTOMER}_$(date +"%Y_%m_%d").psql ${DATABASE};

.. tip::

    In order to create a dumps without binaries, email archive and history, use these arguments additionally::

        --table "nice*" --table "database*" --table "_nice*" --exclude-table-data "nice_email_archive" \
        --exclude-table-data "nice_email_*_to_*" --exclude-table-data "nice_email_attachment" \
        --exclude-table-data "nice_history*"


.. _restore-database:

Restore Database
----------------

.. important::

   Restore must be made into an **empty** database:

   .. parsed-literal::

       CREATE DATABASE **${DB_NAME}** WITH OWNER **${DB_USER}**;

Restore dump file or directory:


    \*.zstd files::

        zstd -qcd ${DUMP_FILE} | pg_restore -U postgres -h ${DB_SERVER} --role ${DB_USER} --no-owner --no-acl -d ${DB_NAME}

    other files and directories::

        pg_restore -j 4 -U postgres -h ${DB_SERVER} --role ${DB_USER} --no-owner --no-acl -d ${DB_NAME} ${DUMP_FILE_OR_DIRECTORY}

.. hint::

    For the restore commands above, You'll need the password for *postgres*  The current password can be found in the
    `tocco_hieradata repository`_ (key: ``profile_postgresql::server::password``. If you're lacking access, ask
    someone with the appropriate permissions for it.


Copy database using WITH TEMPLATE
---------------------------------

This is the fastest way to copy a database. Alternatively, you can dump and then restore the database.

.. parsed-literal::

    CREATE DATABASE **${TARGET_DB}** WITH TEMPLATE **${SOURCE_DB}**;

.. warning::

    This requires that no one is connected to the database. Consequently, it isn't possible to copy a database of
    a running system.

Example
^^^^^^^

This example assumes that the customer name is *tocco* and DB name *nice2_tocco*.

#. switch to the right project

    .. code-block:: bash

        oc project toco-nice-${INSTALLATION}

#. check how many instance are running

    .. code-block:: bash

        oc get dc/nice -o go-template='{{.spec.replicas}}{{"\n"}}'

#. stop instance (if required)

    .. code-block:: bash

        oc scale --replicas=0 dc/nice

#. copy database

    .. parsed-literal:: sql

        CREATE DATABASE **${NAME_OF_DB_COPY}** WITH TEMPLATE **${SOURCE_DB_NAME}**;

    .. hint::

        If you get "source database '…' is being accessed by other users", try :ref:`killing the connections to the
        database <force-close-db-connection>` first.

    .. note::

        By convention, databases not used by a test or production systems should follow this naming pattern:
        ``nice_${CUSTOMER}_${YOUR_SHORT_NAME}_${YEAR}${MONTH}${DAY}``

5. restart instances (if previously stopped)

    .. parsed-literal:: bash

        oc scale --replicas=\ **${N}** dc/nice

    Start **${N}** instances.


.. _tocco_hieradata repository: https://git.vshn.net/tocco/tocco_hieradata/blob/master/database.yaml
