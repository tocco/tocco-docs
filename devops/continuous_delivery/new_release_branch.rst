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
