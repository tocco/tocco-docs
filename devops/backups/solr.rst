Solr Backups
============

Backup
======

Full LVM snapshots are created daily. The backup service is managed by
VSHN. See :vshn:`TOCO-191`.

Restore
=======

List available backups:

.. parsed-literal::

    $ sudo burp -Q -a list
    Backup: **0000197** 2019-09-18 02:26:26 +0200 (deletable)
    Backup: **0000204** 2019-09-24 23:13:16 +0200 (deletable)
    Backup: **0000211** 2019-09-30 22:09:00 +0200 (deletable)
    Backup: **0000218** 2019-10-07 22:15:59 +0200 (deletable)
    Backup: **0000222** 2019-10-11 22:01:23 +0200 (deletable)
    Backup: **0000223** 2019-10-12 23:57:13 +0200 (deletable)
    Backup: **0000224** 2019-10-13 22:19:57 +0200 (deletable)
    Backup: **0000225** 2019-10-14 22:01:03 +0200 (deletable)
    Backup: **0000226** 2019-10-15 22:26:48 +0200 (deletable)
    Backup: **0000227** 2019-10-16 22:12:17 +0200 (deletable)
    Backup: **0000228** 2019-10-18 01:13:12 +0200 (deletable)
            ^^^^^^ **${archive_id}**

List available Solr cores:

.. parsed-literal::

    $ sudo burp -Q -a list -b **${archive_id}** -r '^/var/lib/lvm-snapshots/var-lib-solr-data/[^/]+$'
    ...
    /var/lib/lvm-snapshots/var-lib-solr-data/**nice-tlc**
    /var/lib/lvm-snapshots/var-lib-solr-data/**nice-tlc216**
    /var/lib/lvm-snapshots/var-lib-solr-data/**nice-tlc216test**
    /var/lib/lvm-snapshots/var-lib-solr-data/**nice-tlctest**
    /var/lib/lvm-snapshots/var-lib-solr-data/**nice-tocco**
    /var/lib/lvm-snapshots/var-lib-solr-data/**nice-toccotest**
                                             ^^^^^^^^^^^^^^ **${core_name}**

Restore a specific core:

.. parsed-literal::

    mkdir /tmp/restore
    cd /tmp/restore
    burp -Q -a restore -b **${archive_id}** -r '^/var/lib/lvm-snapshots/var-lib-solr-data/**${core_name}**/' -d .


Stop Solr::

    sudo systemctl stop solr

.. warning::

    This will stop Solr completely. Thus, search requests for **all** cores on the respective server will fail.

Rename current core:

.. parsed-literal::

    sudo mkdir -p /var/lib/solr/data_old
    sudo mv /var/lib/solr/data/nice-\ **${core_name}** /var/lib/solr/data_old/nice-**${core_name}**.old

.. warning::

    Do not just rename the core to \*.old, move it to a different directory as instructed above. If there
    are two copies of the same core in the ``data/`` directory, Solr won't be able to initialize the core.

Replace core with snapshot from backup:

.. parsed-literal::

    sudo mv var/lib/lvm-snapshots/var-lib-solr-data/nice-\ **${core_name}** /var/lib/solr/data/

Start Solr again::

    sudo systemctl start solr

Check status::

    sudo systemctl status solr

Remove old core once no longer needed:

.. parsed-literal:

    rm -rf /var/lib/solr/data_old/**${core_name}**.old
