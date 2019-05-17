Delete Tocco System
^^^^^^^^^^^^^^^^^^^

Delete System on "Nine.ch"
==========================

#. Remove monitoring.

#. Remove in the installation from``~tocco/manager/etc/manager.xml``.

#. Reload config with mgrctl restart.

#. Remove DNS (https://cockpit.nine.ch).

#. Rename the databases::

    ALTER DATABASE nice2_${INSTALLATION} RENAME TO del_nice2_${INSTALLATION}
    ALTER DATABASE nice2_${INSTALLATION}_history RENAME TO del_nice2_${INSTALLATION}_history

#. Wait at least one day for the automatic backup to run.

#. Now you can delete the renamed databases ``del_...``.

#. Remove the project of installation in Teamcity.

#. Update status on the installation to "veraltet" in the Tocco BackOffice.

#. Set the customer module that is linked to the installation to "Obsolete"

#. Delete the Customer's Maven module from the Nice2-Git repository (if there are no other installations that require the Customer module)



Delete Client-System on "VSHN"
==============================

#. Remove monitoring

#. Remove DNS

#. Cloudscale: scale the project to be deleted to 0 instanzes (``oc scale --replaces 0 dc/nice``).

#. Remove the database from `the puppet config <https://git.vshn.net/tocco/tocco_hieradata/blob/master/database/master.yaml>`__

#. Rename the databases::

    ALTER DATABASE nice_${INSTALLATION} RENAME TO del_nice_${INSTALLATION}
    ALTER DATABASE nice_${INSTALLATION} RENAME TO del_nice_${INSTALLATION}

#. Wait at least one day for the automatic backup to run.

#. Now you can access the rename databases ``del_....`` delete.

#. Remove the project of installation in Teamcity.

#. Update status on the installation to "veraltet" in the Tocco BackOffice.

#. Set the customer module that is linked to the installation to "Obsolete"

#. Delete the Customer's Maven module from the Nice2-Git repository (if there are no other installations that require the Customer module)
