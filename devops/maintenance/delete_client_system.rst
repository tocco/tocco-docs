Delete Tocco Installation
#########################

Delete Installation at Nine
===========================

.. _delete-installation-clean-up-app-server:

Clean Up App Server
^^^^^^^^^^^^^^^^^^^

#. Remove the installation from ``~tocco/manager/etc/manager.xml``.

#. Reload config with mgrctl restart.

#. Remove configuration from ``/etc/nginx/sites-enabled/*.conf`` and reload nginx (``nginx -s reload``)

#. Remove Let's Encrypt certificate configuration on app03 at ``/etc/letsencrypt/renewal/*.conf``

#. Remove installation directory

   On app server::

       rm -rf ~/nice2/${INSTALLATION}/


Everything Else
^^^^^^^^^^^^^^^

#. Remove monitoring (Nagios).

#. Remove DNS (https://cockpit.nine.ch).

   Username/password in :term:`secrets2.yml`.

#. Remove the project of installation in Teamcity.

#. Update status on the installation to "veraltet" in the Tocco BackOffice.

#. Set the customer module that is linked to the installation to "Obsolete" if all associated installations are *obsolete*.

   TQL finding all customers with only obsolete installations::

       relModule_status.unique_id != "outdated"
         and relModule_type.unique_id == "customer_module"
         and exists(relInstallation)
         and not exists(relInstallation where relInstallation_status.unique_id != "obsolete")

#. Delete the customer's Maven module from the Nice2 Git repository (if there are no other installations that require the Customer module)

#. Remove Solr index

   Only required when *solr_server* is set in *config.yml*.

   Obtain password to access Solr::

       cd ${ANSIBLE_GIT_REPO}/tocco
       ansible-vault view secrets2.yml |grep solr

   On solr server (e.g. solr2.tocco.cust.vshn.net):

   .. parsed-literal::

       curl 'https\://localhost:8983/solr/admin/cores?action=UNLOAD&deleteInstanceDir=true&core=nice-\ **${INSTALLATION}**\ ' --insecure -u tocco -p

#. Drop databases:

   .. warning::

       Consider waiting a few days before removing the databases to ensure
       the final states of the databases have been backed up.

   .. code::

       DROP DATABASE nice2_${INSTALLATION};
       DROP DATABASE nice2_${INSTALLATION}_history;

#. Remove S3 bucket

   .. warning::

       Consider waiting a few days before removing the S3 bucket to ensure
       the final state of the S3 bucket has been backed up.

   **Only do this if there is no other installation left for the customer.** Buckets
   are shared among all installations of a customer.

   .. code::

       s3cmd rm -rf s3://tocco-nice-${CUSTOMER}
       s3cmd rb s3://tocco-nice-${CUSTOMER}


Delete Installation at VSHN
===========================

See :doc:`/devops/app_management/remove_customer`
