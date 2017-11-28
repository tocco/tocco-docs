Add Customer / Installation
===========================

Create a new Customer
---------------------

.. figure:: add_customer_or_installation_static/new_customer.png

1. Go to the `Continuous Delivery project settings`_
2. **Create subproject** (screenshot above)

   .. _Continuous Delivery project settings: https://dev.tocco.ch/teamcity/admin/editProject.html?projectId=ContinuousDeliveryNg

.. figure:: add_customer_or_installation_static/new_customer2.png

   Project setting for a new customer

3. Fill in parameters as shown above


.. _create-installation-in-teamcity:

Create a new Installation
-------------------------

.. figure:: add_customer_or_installation_static/new_installation1.png

1. Go to the `Continuous Delivery project settings`_
2. Find the customer you want and click on **edit**. If doesn't exist, it needs to be
   `created <#create-a-new-customer>`_ first.

.. figure:: add_customer_or_installation_static/new_installation2.png

   Build configurations for customers

3. **Create build configuration** (screenshot above)

.. figure:: add_customer_or_installation_static/new_installation3.png

   Template parameters

4. Fill in parameters as shown above
5. Fill in these additional template parameters:

   ============================  ============================================================================================
   CUSTOMER                      Customer name (e.g. agogis or smc but never :strike:`agogistest` or :strike:`smctest`)
   DOCKER_PULL_URL               Only needed for production systems. Leave it empty for test system, so a new image is built.
   DUMP_MODE                     ``dump`` for production systems and ``no_dump`` for test systems
   GIT_TREEISH                   Git branch for test systems (e.g. ``releases/2.13``) and empty for production
   INSTALLATION                  Installation name (e.g. smc or smctest)
   ============================  ============================================================================================

   It shouldn't be necessary to touch any of the other parameters.

.. important::

    The installation needs also to be :doc:`created in OpenShift <../openshift/create_nice_installation>`.
