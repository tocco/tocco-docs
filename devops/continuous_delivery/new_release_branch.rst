Create a new release branch
===========================

Create databases
----------------
#. Copy db refactoring_existing from master

   .. parsed-literal::

         CREATE DATABASE dbrefactoring_existing\_\ **${NEW_VERSION}** WITH TEMPLATE dbrefactoring_existing_master;

#. Copy local developer db from master

   .. parsed-literal::

         CREATE DATABASE nice2_test\_\ **${NEW_VERSION}** WITH TEMPLATE nice2_test_master;

.. note::

   Kill database connections if necessary

   .. parsed-literal::

         SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE pg_stat_activity.datname = '**${DB_NAME}**' AND pid <> pg_backend_pid();


Create branches
---------------
- Gerrit -> Projects -> List -> nice2
- Gerrit -> Projects -> Branches

#. Branch Name: releases/**${NEW_VERSION}**, Initial Revision: master
#. Branch Name: integration/releases/**${NEW_VERSION}**, Initial Revision: releases/**${NEW_VERSION}**

Create new configs for db refactoring
-------------------------------------
Teamcity -> Administration -> Nice2 DB-Refactoring

- Copy *Nice2 DB-Refactoring new Database (master)*
- Copy *Nice2 DB-Refactoring new existing Database (master)*
- adjust Parameters

Create new auto merge
---------------------
- Create integration/releases/**${NEW_VERSION}** -> releases/**${NEW_VERSION}**
- Create releases/**${NEW_VERSION}** -> integration/master
- Rename releases/**${LAST_VERSION}** -> integration/releases/**${NEW_VERSION}**
- Adjust parameters in all build configs accordingly

Create new test system
----------------------
- Copy configuration from last version of Continuous Delivery
- Adjust parameters, :ref:`triggers <trigger-deployments>`

Change version in files
-----------------------
Change version in these files in the **master** branch:

- *current-version.txt*
- *web/core/module/resources/webapp/js/version.js*
- Example commit: Idc3f9d9783065ded2079c394db44572802c67c95

Change the database of the test customer in the created **release** branch:

- Change test customer db to **${NEW_VERSION}**: *customer/test/etc/hikaricp.properties*
- Example commit: Ifae0bf7c18ebe0f9810e898c1bca747dc30da0bd

    .. warning::

      The commit will automatically be merged into master and needs to be reverted there.

Sonar
-----
Teamcity:

- Copy config from last version
- Adjust parameters
- Start manually

Sonar:

- Quality Gates -> copy Tocco
- Copy values from last version ->  (*Blocker Issues*, *Critical Issues*)
- Administration -> Projects -> Management -> Create Project

Backoffice
----------
- Change branch of **${LAST_VERSION}**
- Add new Version
- Set status of versions older than 6 versions to outdated
- Check on all installations if **${NEW_VERSION}** is set

Documentation
-------------
- Create a new releases branch in gerrit on the project **«nice2_documentation»**. Use the Revision of the latest version branch as initial revision for the new branch.
- Add a build config for the new version in Teamcity. Use the template **«nice_documentation_allversions»** to create it.
- Add the DNS entry for the new version ${VERSION}.docs.tocco.ch. DNS is available under cockpit.nine.ch (user:tocco/pw:standard-old).
- Create all files needed for Openshift to deploy the new version. You can find a template in the openshift directory in the ansible repository.

     .. parsed-literal::
   
	cd ${PATH_TO_ANSIBLE}/openshift/

	oc login #enter you user name und you password as it will be prompted

	oc project toco-nice-documentation
 
        oc process -f nice-documentation.yml INSTALLATION=${VERSION} | oc create -f -

- Site Search can be configured on http://control.freefind.com and is registered by toccosupport@gmail.com for https://documentation.tocco.ch. Please contact Peter Gerber or Niklaus Hug to get the password.
      
.. attention::
 
   You need the right permissions to create the branch in gerrit and the build config in Teamcity.
