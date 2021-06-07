.. _releasing-script:

Releasing
==========

Semantic Releases
-----------------
A package version consists of three numbers MAJOR.MINOR.PATCH.

Increment as follows:

====== ============================
MAJOR  Will be zero for the moment
MINOR  If new features are added
PATCH  For bug fixes
====== ============================


Release package
---------------
.. warning::

  Before you start, make sure:

  - You checked out the master or a nice-release/* branch. A package should not be released on a feature branch.
  - You're logged into the NPM registry with the "tocconpm" user (check with ``npm whoami``).
    See :ref:`npm-authentication` below for details.

Once the package is ready it can be released / published.

.. code-block:: console

  yarn release-package {PACKAGE_NAME}

This will run yarn setup at the beginning to make sure all local dependencies are up to date.
After that the new version can be entered. The script will then create a changelog entry which can be edited manually,
bumps the version of the package, creates a git release tag and uploads the package to the npm registry.
Everything gets committed and pushed to a new branch with the name of the release. To merge this changes to master or nice-release branch a pull request must be created.
To keep tags intact, the pull request must be closed with "Create a merge commit" and **not** "Rebase and merge". 

.. warning::

  **tocco-resource-scheduler** packages needs a license that is not committed with the code base. This license needs to be
  added prior to releasing.


To check the upcoming changelog:

.. code-block:: console

  yarn changelog {PACKAGE_NAME}

.. _npm-authentication:

Configure npm authentication
------------------------------
We use the default `NPM Registry`_ to upload the packages. All developers use the same user on npm (user "tocconpm" with
email npm@tocco.ch). To setup the authentication execute following command:

.. code-block:: console

  npm adduser

The password can be found on our Sharepoint.

.. _NPM Registry: https://www.npmjs.com/
