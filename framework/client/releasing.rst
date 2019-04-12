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
Once the package is ready it can be released / published. A package will be released on master only.

.. code-block:: console

  yarn release-package {PACKAGE_NAME}

This will run yarn setup at the beginning to make sure all local dependencies are up to date. 
After that the new version can be entered. The script will then create a changelog entry which can be edited manually,
bumps the version of the package, creates a git release tag and uploads the package to the npm registry.
Everything gets committed but needs to be pushed to origin.

.. warning::

  **tocco-resource-scheduler** packages needs a license that is not committed with the code base. This license needs to be 
  added prior to releasing. 


To check the upcoming changelog:

.. code-block:: console

  yarn changelog {PACKAGE_NAME}


Configure npm authentication
------------------------------
We use the default `NPM Registry`_ to upload the packages. All developers use the same user on npm (npm@tocco.ch).
To setup the authentication execute following command:

.. code-block:: console

  npm adduser 

The password can be found on our Sharepoint.

.. _NPM Registry: https://www.npmjs.com/ 