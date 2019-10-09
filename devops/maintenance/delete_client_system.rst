Delete Tocco System
^^^^^^^^^^^^^^^^^^^

Delete System on "Nine.ch"
==========================

#. Remove monitoring (Nagios).

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

#. Set the customer module that is linked to the installation to "Obsolete" if all associated installations are *obsolete*.

   TQL finding all customers with only obsolete installations::

       relModule_status.unique_id != "outdated"
         and relModule_type.unique_id == "customer_module"
         and exists(relInstallation)
         and not exists(relInstallation where relInstallation_status.unique_id != "obsolete")

#. Delete the Customer's Maven module from the Nice2-Git repository (if there are no other installations that require the Customer module)

#. Remove configuration from ``/etc/nginx/sites-enabled/*.conf`` and reload nginx (``nginx -s reload``)

#. Remove Let's Encrypt certificate configuration on app03 at ``/etc/letsencrypt/renewal/*.conf``



Delete Client-System on "VSHN"
==============================

#. Remove monitoring (Nagios)

#. Remove monitoring (VSHN). Remove configuration in `common.yml`_

#. Remove DNS

#. Cloudscale: scale the project to be deleted to 0 instances (``oc scale --replicas 0 dc/nice``).

#. Remove the database from `the puppet config <https://git.vshn.net/tocco/tocco_hieradata/blob/master/database/master.yaml>`__

#. Rename the databases::

    ALTER DATABASE nice_${INSTALLATION} RENAME TO del_nice_${INSTALLATION}
    ALTER DATABASE nice_${INSTALLATION} RENAME TO del_nice_${INSTALLATION}

#. Wait at least one day for the automatic backup to run.

#. Now you can delete the renamed databases ``del_....``.

#. Remove the project of the installation in Teamcity.

#. Set the status on the installation to "veraltet" in the Tocco BackOffice.

#. Set the customer module that is linked to the installation to "Obsolete" if all associated installations are *obsolete*.

   TQL finding all customers with only obsolete installations::

       relModule_status.unique_id != "outdated"
         and relModule_type.unique_id == "customer_module"
         and exists(relInstallation)
         and not exists(relInstallation where relInstallation_status.unique_id != "obsolete")

#. Delete the Customer's Maven module from the Nice2-Git repository (if there are no other installations that require the Customer module)

#. Remove Solr index by changing *state* to *absent* in `solr.yml`_.


.. _common.yml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
.. _solr.yml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/infrastructure/solr.yaml
