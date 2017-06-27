Add Customer / Installation
===========================

Create a new Customer
---------------------

.. figure:: new_customer.png

1. Go to the `Continuous Delivery project settings`_
2. Manually create a new subproject (screenshot above)

   .. _Continuous Delivery project settings: https://dev.tocco.ch/teamcity/admin/editProject.html?projectId=ContinuousDeliveryNg

.. figure:: new_customer2.png

   Project setting for a new customer

3. Fill in parameters as shown above


.. _create-installation-in-teamcity:

Create a new Installation
-------------------------

.. figure:: new_installation1.png

1. Go to the `Continuous Delivery project settings`_
2. Find the customer you want and click on **edit**. If doesn't exist, it needs to be
   `created <#create-a-new-customer>`_ first.

.. figure:: new_installation2.png

   Build configurations for customers

3. Manually create a new subproject (screenshot above)

.. figure:: new_installation3.png

   Template parameters

4. Fill in parameters as shown above
5. Fill in these additional template parameters:

   ============================  ======================================================================================
   CUSTOMER                      Customer name (e.g. agogis or smc but never :strike:`agogistest` or :strike:`smctest`)
   DOCKER_IMAGE_PULL_TAG [#f1]_  ``test`` for **production** systems and empty for test systems
   DOCKER_IMAGE_PUSH_TAG         ``test`` for test systems and ``production`` for production systems
   DUMP_MODE                     ``dump`` for production systems and ``no_dump`` for test systems
   GIT_TREEISH [#f1]_            Git branch for test systems (e.g. ``releases/2.13``) and empty for production
   INSTALLATION                  Installation name (e.g. smc or smctest)
   ============================  ======================================================================================

   It shouldn't be necessary to touch any of the other parameters.

.. important::

    The installation needs also to be :doc:`created in OpenShift <../openshift/create_nice_installation>`.

.. rubric:: Footnotes

.. [#f1] Only one of DOCKER_IMAGE_PULL_TAG and GIT_TREEISH can be used at the same time.

         Generally:

         * GIT_TREEISH is used for test systems, it instructs :term:`CD` to build a new Docker image from source.
         * DOCKER_IMAGE_PULL_TAG is used for production systems and it is set to ``test`` which instruct CD to reuse
           the Docker image currently in use by the test system.
