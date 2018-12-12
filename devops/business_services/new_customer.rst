Create new Customer
===================

Add Customer module
-------------------

:ref:`add_customer_module`

Create Database/Storage
-----------------------

Postgres Database
^^^^^^^^^^^^^^^^^

Add the new databases to the `VSHN git Repository`_.
The database will automatically be created after a few minutes.

.. _VSHN Git Repository: https://git.vshn.net/tocco/tocco_hieradata/edit/master/database/master.yaml

| Use the same password for both databases of the installation
| This password is also used to setup OpenShift.
| Generate a db password:

.. code-block:: bash

  pwgen -s 20 1

Main Database:

.. code-block:: text

  nice_${CUSTOMER}:
    db_user: 'nice_${CUSTOMER}'
    db_password: '${DB_PASS}'
    extensions:
      - 'lo'
      - 'uuid-ossp'


History Database (only version 2.19 and higher):

.. code-block:: text

  nice_${CUSTOMER}_history:
    db_user: 'nice_${CUSTOMER}'
    db_password: '${DB_PASS}'
    extensions:
      - 'lo'

S3 Bucket
^^^^^^^^^

If the customer uses S3, ask operations to create a customer bucket.

Create in OpenShift
--------------------

:ref:`new-installation-openshift`

Create in TeamCity
-------------------

:ref:`new-installation-cd`

Final stepes
------------

#. Setup monitoring

        Setup monitoring as described in the section "Nagios Monitoring einrichten" in
        `this document <https://www.tocco.ch/intranet/Tocco-Workspace/prozesse#detail&key=301&name=Einrich  ten%20einer%20Kundeninstallation>`__.
#. Check installation entry in backoffice.
