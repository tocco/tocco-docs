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
  * adjust parameter *BRANCH*: for v2.25 set it to *releases/2.25*
  * adjust parameter *DATABASE*: for v2.25 set it to *dbrefactoring_new_225*
- Copy *Nice2 DB-Refactoring new existing Database (master)*
  * adjust parameter *BRANCH*: for v2.25 set it to *releases/2.25*
  * adjust parameter *DATABASE*: for v2.25 set it to *dbrefactoring_existing_225*

Create new auto merge
---------------------
- Create integration/releases/**${NEW_VERSION}** -> releases/**${NEW_VERSION}**
- Create releases/**${NEW_VERSION}** -> integration/master
- Rename releases/**${LAST_VERSION}** -> integration/releases/**${NEW_VERSION}**
- Adjust parameters in all build configs accordingly

Create new test system
----------------------
- Create new test system :doc:`via Ansible </devops/app_management/new_customer>`
- Adjust parameters, :ref:`triggers <trigger-deployments>`

Remove old test system(s)
-------------------------

Keep *master* and the most recent 7 test systems (all supported versions plus the just
created test system) and remove any older test system.

See :doc:`/devops/app_management/remove_customer`

Update internal SSO App
^^^^^^^^^^^^^^^^^^^^^^^

1. Login to Microsoft Azure https://portal.azure.com/
2. Open the ``Tocco intern SSO`` app (Open ``App registrations`` service via search and find item in list)
3. Open ``Authentication`` tab
4. Add new ``Redirect URI`` for the created test system  (e.g. https://test225.tocco.ch/nice2/sso-callback for ``test225``)
5. Save

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

Create release branch in tocco-client repository
------------------------------------------------
Head over to the `tocco-client Repository`_ and create a new release branch based on the current master revision.
Replace **${VERSION}** with the version number without any characters which aren’t numeric (e.g. 2.18 -> 218).

.. _tocco-client Repository: https://github.com/tocco/tocco-client

.. parsed-literal::

   git checkout -b nice-releases/**${VERSION}** && git push

Afterwards checkout master again and replace the nice version inside the file `nice-current-version`_ . 
This change must be committed and pushed and a pull-request should be opened.

.. _nice-current-version: https://github.com/tocco/tocco-client/blob/master/nice-current-version.txt

TeamCity build configs in `developer dashboard`_
------------------------------------------------

Head over to the `tocco-dashboard Repository`_ and add the new build configs you created for the new release branch
(for the DB refactorings, auto merge and for the deployment of the new test instance) in the file `apiCalls.js`

Example commit: `30a39e1`_

.. _developer dashboard: https://dashboard.tocco.ch
.. _tocco-dashboard Repository: https://github.com/tocco/tocco-dashboard
.. _30a39e1: https://github.com/tocco/tocco-dashboard/commit/30a39e1a72607c56156365a61f90ea8a796c7c17

New Sonar project
-----------------

Teamcity
^^^^^^^^

- Copy config from last version
- Adjust parameters (*git-branch-name* and *sonar-branch*)
- Start manually (this will create the project in Sonar automatically)

Sonar
^^^^^

    .. warning::

      These tasks require Sonar admin rights.

- Enter menu Quality Gates
- Copy *Tocco Default* with new version name
- Set values to those from the current analysis -> (*Blocker Issues*, *Critical Issues* and *Coverage*)
- Connect the copied Quality Gate to the newly created project at the bottom

Backoffice
----------
- Change branch of **${LAST_VERSION}**
- Add new Version
- Set status of versions older than 6 versions to outdated (on release date)
- Check on all installations if **${NEW_VERSION}** is set

Create tasks
------------

To update outdated Maven dependencies
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It's important to keep external dependencies up to date and it makes sense to update them at the very beginning
of a release development cycle (to be able to spot problems early during the development cycle).

Therefore, **create a task** to update the outdated dependencies in one of the first sprints.

See chapter :ref:`update_dependencies_on_a_regular_basis` to learn where you get the list of outdated dependencies from.

To update Hibernate documentation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A new JIRA task should be created to keep the Hibernate documentation up to date.
All changes in the ``persist/core`` module since the last release should be reviewed
and the documentation should be adjusted if necessary.

For `toccotest.tocco.ch`_ migration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`toccotest.tocco.ch`_ should be migrated to the new version as soon as possible after this branch has been created.
This is done by the Tocco Dev team (not by the Business Services).

A new JIRA task should be created in the `TOCBO`_ project and assigned to the Dev team.

.. _TOCBO: https://toccoag.atlassian.net/projects/TOCBO
.. _toccotest.tocco.ch: https://toccotest.tocco.ch

For `www.tocco.ch`_ migration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Approximately one week before the release date, our Tocco Backoffice should be updated to the new version.
This is done by the Tocco Business Services.

A new JIRA task should be created in the `TOCBO`_ project and assigned to the Business Services team.

.. _www.tocco.ch: https://www.tocco.ch

For `demo.tocco.ch`_ migration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Approximately one week before the release date, our demo installation `demo.tocco.ch`_ should be updated to the new
version. This is done by the Tocco Business Services.

A new JIRA task should be created in the `TOCBO`_ project and assigned to the Business Services team.

.. _demo.tocco.ch: https://demo.tocco.ch

For `integration.tocco.ch`_ migration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Approximately one week before the release date, our integration installation `integration.tocco.ch`_ should be updated to the new
version. This is done by the Tocco Business Services.

A new JIRA task should be created in the `TOCBO`_ project and assigned to the Business Services team.

.. _integration.tocco.ch: https://integration.tocco.ch

Store entity model snapshot on SharePoint
-----------------------------------------

On the *release date* (not when the release branch is created), the current entity model snapshot should be obtained
from the test system of the new version and stored on our SharePoint.

#. Get the JSON snapshot from: https\://test\ **${VERSION}**.tocco.ch/nice2/rest/entities?_fullModel=true&_omitLinks=true
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
- Add a build config to the project **Nice2 Documentation** for the new version in Teamcity. Use the template
  **«nice_documentation_allversions»** to create it.
- Add a build config to the project **Nice2 Specification** for the new version in Teamcity. Use the template
  **«nice2_specification»** to create it.
- Run the first build in TeamCity. Please note that ${VERSION}.docs.tocco.ch won't serve any content before the first
  build has completed.
- Add the DNS entry for the new version ${VERSION}.docs.tocco.ch. DNS is available under https://cockpit.nine.ch.
  (username/password in :term:`secrets2.yml`.)
- Create all files needed for Openshift to deploy the new version. You can find a template in the openshift directory
  in the ansible repository. Replace **${VERSION}** with the version number without any characters which aren't numeric
  (e.g. 2.18 -> `218`).

  .. parsed-literal::

      cd ${PATH_TO_ANSIBLE}/openshift/
      oc login #enter you user name und you password as it will be prompted
      oc project toco-nice-documentation
      oc process -f nice-documentation.yml INSTALLATION=${VERSION} | oc create -f -

- Issue TLS certificate::

      oc annotate route/documentation-${VERSION} kubernetes.io/tls-acme=true

  Here again, ${VERSION} is *218* rather than *2.18*.

- Site Search can be configured on https://control.freefind.com and is registered by toccosupport@gmail.com for
  https\://documentation.tocco.ch. Username and password can be found in :term:`secrets.yml`.

  1. Set an additional starting point in "/Build Index/Set starting point" to ensure that the subdomain is indexed.
  2. Define a new subsection in "/Build Index/Define subsections" to ensure that user can search inside a specific documentation.
  3. Restart indexing immediately by "/Build Index/Index now".

.. attention::

   You need the right permissions to create the branch in gerrit and the build config in Teamcity.

Troubleshooting
^^^^^^^^^^^^^^^

If SSL doesn't work correctly, make sure TLS integration has been enabled (See :ref:`ssl-cert-issuance`).

Standard specification
----------------------

The standard specification is part of the **«nice2_documentation»** project and needs its own build config in TeamCity.

Therefore, add a build config for the new version in Teamcity like you did for the documentation. Copy the build config
for the previous version in the project **«Nice2 Specification»** and adjust the parameters accordingly.

Setup Monitoring
----------------

Enable monitoring for the documentation in `common.yml`_. Look for for *docs.tocco.ch*.

Check for Unused Modules
------------------------

Go to `Unused Dependencies`_ in TC and generate a new report (*Run* button). Wait
for the build to complete. Then go to the *Artifacts* tab on the result page and
open *result.txt*.

Check for unused modules and have them removed. Note that modules can be listed
as unused if they are new and not yet used.
 

.. _common.yml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
.. _Unused Dependencies: https://tc.tocco.ch/buildConfiguration/Nice2_UnusedDependencies

Update Initial Values
---------------------

Run the scripts as explained in section :ref:`initial-values` to update the initial values in the *integration/master* branch.
