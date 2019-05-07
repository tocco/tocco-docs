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

#. Copy local customer history db from master

   .. parsed-literal::

         CREATE DATABASE nice2_test\_\ **${NEW_VERSION}**\_\history WITH TEMPLATE nice2_test_master_history;

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
- Change test customer history db to **${NEW_VERSION}**: *customer/test/etc/hikaricp.history.properties*
- Example commits: Ifae0bf7c18ebe0f9810e898c1bca747dc30da0bd, 9a2f3bb9937cc5ed39efa66fa13e458a64b3c45d

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
- Set status of versions older than 6 versions to outdated (on release date)
- Check on all installations if **${NEW_VERSION}** is set

Store entity model snapshot on SharePoint
-----------------------------------------

On the *release date* (not when the release branch is created), the current entity model snapshot should be obtained
from the test system of the new version and stored on our SharePoint.

#. Get the JSON snapshot from: https\://test\ **${VERSION}**.tocco.ch/nice2/rest/entities?_fullModel=true
#. Save it as JSON file and put it into the corresponding release directory on our `share point`_. The file should
   be called ``Entity_Model_${VERSION}.json``.

.. _share point: https://tocco.sharepoint.com/:f:/s/Produkt-Gilde/EjCp-srbI5FNmAdoqZ94MRgB3BxJfc8vs0QgIXrVYhvc8A?e=QYThAB

Compare two snapshots to view changes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To view the differences between two model snapshots any text or JSON diffing tool can be used. However, keep in mind
that the snapshot files can be quite big and that some tools might not be able to cope with that.

One tool that works quite well is Meld. Meld is free to use and available for Windows, Linux and MacOS.

Steps to compare two files using Meld:

#. Get Meld from https://meldmerge.org
#. Open Meld and press the button **File comparison**

   .. figure:: compare_entity_models_static/meld1.png

#. **Don't** select the snapshot files yet (leave the file selection fields empty with the placeholder "(None)")
   and press **Compare**.

   .. hint::

     The reason for leaving the file selection fields empty is that Meld isn't able to detect the encoding correctly
     if the files are selected already here.

   .. figure:: compare_entity_models_static/meld2.png

#. Select the old and the new snapshot file at the top of the two columns. Note that it can take two minutes or so to
   load the files in Meld (loading state indicated by loading icon in the top right corner).

   .. figure:: compare_entity_models_static/meld3.png

#. Once both files are loaded, the differences are highlighted and can be spotted easily. Use the arrow buttons to
   navigate between the differences.

   .. figure:: compare_entity_models_static/meld4.png

Documentation
-------------

.. attention::

   You have to clone the ansible repository to access the files mentioned below. You can clone the project with the
   following command: **git clone ssh://${GERRIT_USERNAME}@git.tocco.ch:29418/ansible**

- Create a new releases branch in gerrit on the project **«nice2_documentation»**. Use the Revision of the latest
  version branch as initial revision for the new branch.
- Add a build config for the new version in Teamcity. Use the template **«nice_documentation_allversions»** to create
  it.
- Run the first build in TeamCity. Please note that ${VERSION}.docs.tocco.ch won't serve any content before the first
  build has completed.
- Add the DNS entry for the new version ${VERSION}.docs.tocco.ch. DNS is available under cockpit.nine.ch
  (user:tocco/pw:standard-old).
- Create all files needed for Openshift to deploy the new version. You can find a template in the openshift directory
  in the ansible repository. Replace **${VERSION}** with the version number without any characters which aren't numeric
  (e.g. 2.18 -> `218`).

     .. parsed-literal::

	cd ${PATH_TO_ANSIBLE}/openshift/

	oc login #enter you user name und you password as it will be prompted

	oc project toco-nice-documentation

        oc process -f nice-documentation.yml INSTALLATION=${VERSION} | oc create -f -

- Site Search can be configured on https://control.freefind.com and is registered by toccosupport@gmail.com for
  https\://documentation.tocco.ch. Please contact Peter Gerber or Niklaus Hug to get the password.

  1. Set an additional starting point in "/Build Index/Set starting point" to ensure that the subdomain is indexed.
  2. Define a new subsection in "/Build Index/Define subsections" to ensure that user can search inside a specific documentation.
  3. Restart indexing immediately by "/Build Index/Index now".

.. attention::

   You need the right permissions to create the branch in gerrit and the build config in Teamcity.

Troubleshooting
^^^^^^^^^^^^^^^

If SSL doesn't work correctly, make sure TLS integration has been enabled (See :ref:`ssl-cert-issuance`).
