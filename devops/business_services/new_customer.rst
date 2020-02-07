Create new Customer
===================

Add Customer module
-------------------

:doc:`/framework/configuration/modules/add-customer-module`

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

#. Add a new core in :hierra-repo:`infrastructure/solr.yaml`:

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

#. Setup Monitoring

   See :ref:`monitoring-generate-checks`

#. Check installation entry in backoffice.


.. _common.yaml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
