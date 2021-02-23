Database Backups
================

Get the Latest Backup
---------------------

Databases are dumped to disk on the slave at ``/var/lib/postgresql-backup/`` before they are archived. If you need the most
current backups you can get it from there.

:ref:`restore-database` explains how to restore a dump.


Get Backup Created by :term:`CD`
--------------------------------

Dumps created by checking the option "dump database" in TeamCity are created in ``/var/lib/postgresql-backup/deploy-dumps/``.
Dumps in that directory are removed after a few days. If a dump has been removed already, you'll have to extract it out of the
archive, see below.

:ref:`restore-database` explains how to restore a dump.


Backup Locations
----------------

Production Cluster 1
^^^^^^^^^^^^^^^^^^^^

============================  ========  ==================================  =====================================================
        Server                 Role                 Daily Backups                    Triggered backups (during CD)
============================  ========  ==================================  =====================================================
db1.prod.tocco.cust.vshn.net   master    n/a                                 ``/var/lib/postgresql-backup/deploy-dumps/`` [#f1]_
db2.prod.tocco.cust.vshn.net   slave     ``/var/lib/postgresql-backup/``     ``/var/lib/postgresql-backup/deploy-dumps/`` [#f1]_
============================  ========  ==================================  =====================================================

Production Cluster 2
^^^^^^^^^^^^^^^^^^^^

=======================  ========  ==================================  =====================================================
        Server            Role                 Daily Backups                    Triggered backups (during CD)
=======================  ========  ==================================  =====================================================
db3.tocco.cust.vshn.net   master    n/a                                 ``/var/lib/postgresql-backup/deploy-dumps/`` [#f1]_
db4.tocco.cust.vshn.net   slave     ``/var/lib/postgresql-backup/``     ``/var/lib/postgresql-backup/deploy-dumps/`` [#f1]_
=======================  ========  ==================================  =====================================================

Staging Cluster 1
^^^^^^^^^^^^^^^^^

=============================  ========  ==================================  =====================================================
        Server                  Role                 Daily Backups                    Triggered backups (during CD)
=============================  ========  ==================================  =====================================================
db1.stage.tocco.cust.vshn.net   master    n/a                                 ``/var/lib/postgresql-backup/deploy-dumps/`` [#f1]_
db2.stage.tocco.cust.vshn.net   slave     ``/var/lib/postgresql-backup/``     ``/var/lib/postgresql-backup/deploy-dumps/`` [#f1]_
=============================  ========  ==================================  =====================================================

Get Backup from Archive
-----------------------

The directory ``/var/lib/postgresql-backup/``, which contains dumps made during backups and deployments, are archived
daily using :term:`BURP`. If you need backups/dumps no longer in these directories, you can extract them from
the archive.

.. warning::

      You need root access to access the archive.

.. hint::

   Other directories, like ``/home/``, ``/usr/local/``, … are included in the archive too.


List Available Archives
^^^^^^^^^^^^^^^^^^^^^^^

.. parsed-literal::

      $ sudo burp -c /etc/burp/slave.conf -C **${NAME_OF_SLAVE}** -a list
      Backup: 0000155 2018-01-31 03:03:22 +0100 (deletable)
      Backup: 0000162 2018-02-09 01:14:05 +0100 (deletable)
      Backup: :green:`0000169` 2018-02-16 01:10:54 +0100 (deletable)
      Backup: 0000176 2018-02-23 01:06:28 +0100 (deletable)
      …

.. hint::

   If ``-C ${NAME_OF_SLAVE}`` is not specified, the archives from the current host are
   listed. ``-C`` allows you to restore a dumps made on the slave directly on the master.
   See tables above to find out which slave belong to which master.

List Files in Archive
^^^^^^^^^^^^^^^^^^^^^

Show the content of directory ``/var/lib/postgresql-backup/`` in archive :green:`0000169`.

.. parsed-literal::

      $ sudo burp -c /etc/burp/slave.conf -C **${NAME_OF_SLAVE}** -a list -b :green:`0000169` -r '^/var/lib/postgresql-backup/'
      Backup: 0000169 2018-02-16 01:10:54 +0100 (deletable)
      With regex: ^/var/lib/postgresql-backup/
      /var/lib/postgresql-backup/nice_awpf.dump
      /var/lib/postgresql-backup/nice_awpftest.dump
      :red:`/var/lib/postgresql-backup/nice_bnftest.dump`
      /var/lib/postgresql-backup/nice_dghtest.dump
      /var/lib/postgresql-backup/nice_esrplus.dump


Extract File from Archive
^^^^^^^^^^^^^^^^^^^^^^^^^

Restore **nice_bnftest.dump** from backup :green:`0000169` to directory **~/restores/**.

.. parsed-literal::

      $ mkdir -p :blue:`~/restores/`
      $ sudo burp -c /etc/burp/slave.conf -C **${NAME_OF_SLAVE}** -a restore -b :green:`0000169` -d :blue:`~/restores/` -r '^\ :red:`/var/lib/postgresql-backup/postgres-nice_bnftest.dump.gz`'
      …
      2018-03-09 16:01:30 +0100: burp[23156] restore finished
      $ ls -lh :blue:`~/restores/`:red:`var/lib/postgresql-backup/nice_bnftest.dump`
      -rw-rw-r-- 1 postgres postgres 4.1G Feb 16 01:26 /home/peter.gerber/restores/var/lib/postgresql-backup/nice_bnftest.dump

:ref:`restore-database` explains how to restore a dump.


.. rubric:: Footnotes

.. [#f1] The output of the *dump* step in CD prints the server on which the dump is located as
         well as the path to it on the server.
