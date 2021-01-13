Remove Installation/Customer
============================

Remove OpenShift Project, S3 Bucket, Database, Etc.
---------------------------------------------------

#. Add ``state: absent`` to all installations to be removed:

    .. code-block:: yaml

       definitions:
         # ...
         abc:
           # ...
           installations:
             abc:
               state: absent                         # <-- remove this installation
               db_server: db1.tocco.cust.vshn.net
               solr_server: solr2.tocco.cust.vshn.net
             abctest:
               state: absent                         # <-- remove this installation
               db_server: db1.tocco.cust.vshn.net
               solr_server: solr2.tocco.cust.vshn.net
         # ...

#. Remove installation(s)

   There are two options available:

   a) Remove installation but leave DBs and S3 bucket intact:

      .. parsed-literal::

           cd ${GIT_ROOT}/tocco
           ansible-playbook playbook.yml -l **abc,abctest**

   b) Remove everything, **including DBs and S3 bucket**:

      .. parsed-literal::

           cd ${GIT_ROOT}/tocco
           ansible-playbook playbook.yml -t all,force-irrevocable-removal -l **abc,abctest**

    Note about S3 buckets:

    An S3 bucket may be used by multiple installations and it won't be removed unless
    all remaining users (i.e. installations) are marked with ``state: absent``.

    .. tip::

        Do a dry run using ``--check`` if you want to know what would be removed without
        actually removing anything.


Remove Installation from Inventory
----------------------------------

Remove the installation and possibly customer from ``config.yml`` and commit the change.


Remove DNS Entry
----------------

Go to https://cockpit.nine.ch and remove the DNS record for ${installation}.tocco.ch
and any other records associated with the installation.

Username/Password in :term:`secrets2.yml`.


Update information in :term:`BO`
--------------------------------

* Set the installation status to "Veraltet"
* Set the customer module that is linked to the installation to "Veraltet" if all
  associated installations are obsolete.

  TQL finding all customers with only obsolete installations::

      relModule_status.unique_id != "outdated"
        and relModule_type.unique_id == "customer_module"
        and exists(relInstallation)
        and not exists(relInstallation where relInstallation_status.unique_id != "obsolete")


Remove Maven Module from Nice2 Repo
-----------------------------------

Delete the customer's Maven module from the Nice2 Git repository if there are no other
installations that require the customer module.
