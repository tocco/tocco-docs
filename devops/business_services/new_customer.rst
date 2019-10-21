.. _create_new_customer:

Create new Customer
===================

Add Customer module
-------------------

:ref:`add_customer_module`

Create Database/Storage
-----------------------

Postgres Database
^^^^^^^^^^^^^^^^^

Add the new databases to the :hierra-repo:`VSHN Git Repository <database/master.yaml>`.
The database will be created automatically. This can take up to 40 minutes.

Use the same password for both databases of the installation.

#. Generate a DB password::

       pwgen -s 20 1

   .. hint::

       This password has to be passed to ``oc process`` as ``DB_PASS`` parameter later on.

#. Main Database

   .. code-block:: yaml

       nice_${INSTALLATION}:
         db_user: 'nice_${INSTALLATION}'
         db_password: '${DB_PASS}'
         extensions:
           - 'lo'
           - 'uuid-ossp'


#. History Database

   .. code-block:: yaml

       nice_${CUSTOMER}_history:
         db_user: 'nice_${CUSTOMER}'
         db_password: '${DB_PASS}'


Create a Solr Core
^^^^^^^^^^^^^^^^^^

#. Generate ``${SOLR_PASSWORD}``::

       pwgen -s 30 1

   .. hint::

       This password has to be passed to ``oc process`` as ``SOLR_PASS`` parameter later on.

#. Add a new user and core in :hierra-repo:`infrastructure/solr.yaml`:

   .. code-block:: yaml

       profile_solr::basic_auth_users:
         nice-${INSTALLATION}: ${SOLR_PASSWORD}

   .. code-block:: yaml

       profile_solr::hiera_cores:
         nice-${INSTALLATION}:
           ensure: present


.. todo::

    Comment in the following section once S3 is ready.

..
    Create S3 Bucket
    ^^^^^^^^^^^^^^^^

    See :doc:`/devops/s3/s3_bucket_for_installation`.

Create in OpenShift
--------------------

:ref:`new-installation-openshift`

Create in TeamCity
-------------------

:ref:`new-installation-cd`

Final Steps
------------

#. Setup monitoring

        Setup monitoring as described in the section "Nagios Monitoring einrichten" in
        `this document <https://www.tocco.ch/intranet/Tocco-Workspace/prozesse#detail&key=301&name=Einrich  ten%20einer%20Kundeninstallation>`__.
#. Check installation entry in backoffice.
