Deploy / Basics
===============

Teamcity Project "Continuous Delivery"
--------------------------------------

The `Continuous Delivery project`_ is the entry point to CD.

.. _Continuous Delivery project: https://dev.tocco.ch/teamcity/project.html?projectId=Nice2ContinuousDelivery

.. figure:: deploy_and_basics_static/tc_main.png
   :scale: 60%
   :name: main page

   *Continuous delivery* project on on TC's main page

Deliver (Simple)
----------------

.. note::

   Use `Deliver (Advanced)`_ if you need to deploy …
      #. … a test system with a custom Git branch or tag
      #. … a production system with a Docker image previously installed on the test system

.. figure:: deploy_and_basics_static/tc_run_menu.png		
   :name: run menu

   *Run* menu

#. Click on **Run** in the `main page`_		
#. (optional) adjust the `dump mode`_ in the `run menu <#run-menu>`__
#. Click **Run Build** in the `run menu <#run-menu>`__

Deliver (Advanced)
------------------

.. figure:: deploy_and_basics_static/tc_parameters_tab_with_ellipsis.png
   :name: run menu advanced

   Full **Parameters** menu as shown when opening via ellipsis (...)

.. figure:: deploy_and_basics_static/tc_run_changes_tab.png

   *Changes* tab in *Run* menu

#. Click on **Run** in the `main page`_.
#. (optional) Adjust the `dump mode`_ in the `run menu <#run-menu>`__.
#. (optional) Select a particular `Git tag or branch <#deploy-a-specific-git-tag>`_ or deploy a particular `Docker image
   <#deploy-a-specific-docker-image>`_.
#. Click **Run Build** in the `run menu <#run-menu>`__.


Dump Mode 
---------

.. note:: dump mode is currently not available 

.. figure:: deploy_and_basics_static/tc_dump_modes_dropdown.png

   **Dump Mode** dropdown on **Parameters** tab in **Run** menu

=========================  =============================================================================================
do not dump database       Deploy without creating a dump first (default for test systems.)
dump database              Create a dump and only then deploy (default for production systems.)
dump and restore database  In case of a deployment failure, automatically roll back by restoring the created dump.
                           **In case of a rollback, changes made to the DB, after starting the dump, are lost!**
=========================  =============================================================================================


Deploy a Specific Git Tag
-------------------------

.. figure:: deploy_and_basics_static/tc_changes_tab_dropdown.png

   **Build branch** dropdown on **Changes** tab in **Run** menu

**Build branch** allows you to specify to deploy an arbitrary Git branch or tag.

.. note:: There might be a situation where you want to deploy a tag directly on production. 
          In that case remove the CD parameter "DOCKER_PULL_URL". `See deploy a Specific Docker Image <#deploy-a-specific-docker-image>`_.


Deploy a Specific Docker Image
------------------------------

This is used for production systems, it allows you to deploy a docker image that was once installed on the test
system. The image is determined by the DOCKER_PULL_URL parameter. It allows you to pick a image from a project an deploy it on your target project.
E.g. This deploys the image from toccotest if you're deploying tocco. For this you can fill the parameter with the following: 

**registry.appuio.ch/toco-nice-%env.INSTALLATION%test/%env.DOCKER_IMAGE%.**

After the evaluation of the CD parameters it will appear as an URL like this: 
         
``registry.appuio.ch/tocco-nice-toccotest/nice:latest`` [#f1]_


.. rubric:: Footnotes

.. [#f1] It is also possible to deploy an image from any other system on your target system.
         E.g. test213 -> test214
         After that, test214 will have the state of test213 which obvosuly makes no sense at all.

         So be aware of that, and only use it to deploy a test image on a production system.


