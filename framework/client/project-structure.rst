Project Structure
==================
Monorepo
--------
tocco-client is a menorepo and uses `Lerna`_

.. _Lerna: https://lernajs.io/

All packages are located within the ``packages/`` folder. Some export an independent react app others act as a util package with components and helper functions.
A package that export a self containing react app is called a tocco-app. These tocco-apps, in contrast to the util ones, can be released and are easily identified by the absent `private` flag in the ``package.json``.
Every package maintains its own dependencies and can be re-used in other packages. With all due caution against introducing circular dependencies!
A description of each package can be found in its README.

Package Naming
^^^^^^^^^^^^^^
Please ensure that every package is prefixed with ``tocco-``

``tocco-...`` naming is used in ``package.json``; in folder structure ``tocco-`` prefix can be omitted.


Create New Package
^^^^^^^^^^^^^^^^^^
A plop template can be used that creates the essentials of a new package.

.. code-block:: console

 yarn plop Package {PACKAGE_NAME}


Add local or remote package as dependency:

.. code-block:: console

 lerna add --D react-select --scope {PACKAGE_NAME} --no-bootstrap
 