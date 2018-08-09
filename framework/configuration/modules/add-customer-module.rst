.. |pathToModuleFolder| replace:: ``nice2-project/customer/mynewcustomer``
.. |pathToModuleModuleFolder| replace:: ``nice2-project/customer/mynewcustomer/module/module``
.. |pathToModulePom| replace:: ``nice2-project/customer/mynewcustomer/pom.xml``
.. |pathToModuleModulePom| replace:: ``nice2-project/customer/mynewcustomer/module/module/pom.xml``

Add Customer Module
===================

Adding a new module contains the following steps:

.. list-table::
   :header-rows: 1
   :widths: 10 10 80

   * - Nr
     - Required
     - Description
   * - 1
     - ✔
     - `Add Module in Backoffice`_
   * - 2
     - ✔
     - `Create Basic Folder Structure`_
   * - 3
     - ✔
     - `Add Application Properties`_
   * - 4
     - ✔
     - `Add HikariCP Properties`_
   * - 5
     -
     - `Add Content to module Folder`_
   * - 6
     -
     - `Add Java Source Folders`_
   * - 7
     - ✔ **if 5 is done**
     - `Build Resources into Target Snapshot`_

.. include:: add-module-in-backoffice.rst

Create Basic Folder Structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Assume the new module is called ``mynewcustomer``.

Add a new folder ``mynewcustomer``  inside ``nice2-project/customer`` with the following file structure

  .. figure:: resources/basic-folder-structure-customer.png

Open the file ``mynewcustomer/pom.xml`` and add the following content.

.. literalinclude:: resources/customer-module-pom-1.xml
   :language: XML

Open the file ``customermodule/module/module/pom.xml`` and add the following content.

.. literalinclude:: resources/customer-module-pom-2.xml
   :language: XML

Open the file ``customermodule/module/module/hiveapp-mount.properties`` and add the following content.

.. literalinclude:: resources/customer-module-hiveapp-mount.properties

Open the file ``mynewmodule/module/descriptor/hivemodules.xml`` and add the following content.

.. literalinclude:: resources/customer-module-hivemodule.xml
   :language: XML


Add Application Properties
^^^^^^^^^^^^^^^^^^^^^^^^^^

For each customer module an ``application.properties`` file exists. The content of this file are properties which configure
the customer installation. Create a file ``application.properties`` in ``nice2-project/customer/CUSTOMERNAME/etc`` and
add the following content.

.. literalinclude:: resources/customer-module-application.properties

Add HikariCP Properties
^^^^^^^^^^^^^^^^^^^^^^^

`HikariCP`_ the JDBC connection pool used in Nice2. In order to work correctly with the customer installation, some
database properties must be set. For each customer module a ``hikaricp.properties`` file exists. Create a file
``hikaricp.properties`` in ``nice2-project/customer/CUSTOMERNAME/etc`` and add the following content.

.. literalinclude:: resources/customer-module-hikaricp.properties


Add Content to module Folder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Inside the module folder |pathToModuleModuleFolder| different folders which configure the module can
be added. Here are the most common use cases:

**model**

The model folder must be added as soon as you need to

* add or adjust entities or relations -> see :ref:`Entities and Relations`
* add or adjust text resources -> see :ref:`Text-Resources`
* add or adjust forms (list, search, detail) -> TODO (add reference when chapter is written)
* add or extend a menu (settings or modules) -> see :ref:`Menu`
* add reports -> see :ref:`Reports`

**db**
Inside the db folder changesets are placed. See chapter :ref:`Changesets`.

**acl**
Inside the acl folder all acl rule files are located. See chapter :ref:`acl`

**resources**
Inside the resources folder JS files are placed. For actions and public flows.

**outputtemplate**
Inside this folder ``ftl`` templates are placed which for example can be used for reports.

.. include:: add-java-source-folder.rst

.. include:: build-resources-into-target-snapshot.rst

.. include:: add-module-with-script.rst

.. _HikariCP: https://github.com/brettwooldridge/HikariCP

