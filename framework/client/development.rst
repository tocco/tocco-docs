.. |br| raw:: html

    <br>


Development
===========

Setup
-----

Just install `Yarn`_ and execute the following commands:

.. _Yarn: https://yarnpkg.com/en/docs/install 



.. code-block:: console

  git clone https://github.com/tocco/tocco-client.git
  cd tocco-client
  npm install --global lerna@v3.10.5
  yarn setup


For some packages/components you may need secret keys or passwords you can't commit or save in the codebase. In such cases
you can set your secret environment variables in a ``/.env`` and add documentation to the Plop template.

Enter this command to create your ``/.env`` file:

.. code-block:: console

  yarn plop Env


Start a package
----------------

.. code-block:: console

  yarn start --package={PACKAGE_NAME}


This automatically opens http://localhost:3000 with activated hot reloading.

.. note::

  Hot reloading will allow parts of the application to be live reloaded when the source code changes. 
  Keep in mind that hot reloading will not work for sagas.



Optional parameters


=========================== ============================
``--noMock``                Will disable mocked data. |br| In this case you must run the Tocco Business Framework application |br| with enabled REST API on http://localhost:8080
``--backend={BACKEND_URL}``  Enable an alternative backend. E.g. master deployment of Tocco.
=========================== ============================



Storybook
---------
It might be helpful to start up Storybook locally to test the current state of development. 
Most of the components or packages have a dedicated story to run them isolated.
Storybook can be started with the following 
command:

.. code-block:: console

 yarn storybook


Use ``BACKEND={BACKEND_URL} yarn storybook`` to enable an alternative backend.

Tests
-----

Tests are using following tools and libraries:

* `Jest`_
* `Sinon`_
* `Chai`_
* `Enzyme`_

.. _Jest: https://jestjs.io/
.. _Sinon: http://sinonjs.org/
.. _Chai: http://chaijs.com/
.. _Enzyme: https://github.com/airbnb/enzyme


Run unit tests with Jest

.. code-block:: console

  yarn test

Optional parameters

======================================= ============================
``--projects packages/{PACKAGE_NAME}``   To only run tests of one packages. |br| This will reduce runtime drastically. |br| It's possible to add multiple projects/packages.
``--watch``                              Run jests watch mode
======================================= ============================


.. note::
 If working with IntelliJ single tests or test-suites can be run in the IDE directly. Just set the jest.config.js file in the Jest run configuration. 


Code Generators
---------------
The project provides some code generators. Generators are developed with `Plop`_ and can be executed with:

.. code-block:: console

  yarn plop

At the moment there is a generator to create a react-component, to add a redux-action, to create a package
and to initiate a .env file with your environment keys.

.. _Plop: https://github.com/amwmedia/plop


Code Styleguide
-----------------------------------------
See :ref:`Coding-Styleguide` 

Build bundle
------------
Sometimes it's desired to only build a package for testing purposes.

.. code-block:: console

    yarn compile:dev --package={PACKAGE_NAME}
    yarn compile:prod --package={PACKAGE_NAME}

Parameters

=========================== ============================
``--bundle-analyzer``        Opens BundleAnalyzerPlugin to investigate the bundle sizes.
``--backend={BACKEND_URL}``  To enable an alternative backend.
=========================== ============================

