.. |pathToModuleFolder| replace:: ``path/to/nice2/customer/mynewcustomer``
.. |pathToModuleModuleFolder| replace:: ``path/to/nice2/customer/mynewcustomer/module/module``
.. |pathToModulePom| replace:: ``path/to/nice2/customer/mynewcustomer/pom.xml``
.. |pathToModuleModulePom| replace:: ``path/to/nice2/customer/mynewcustomer/module/module/pom.xml``
.. |pathToImplFolder| replace:: ``path/to/nice2/customer/mynewcustomer/module/impl``
.. |pathToImplPom| replace:: ``path/to/nice2/customer/mynewcustomer/module/impl/pom.xml``

Add Customer Module
===================

.. hint::
   This chapter describes how to add a new customer module manually. There is also a script which can be used to
   generate the module. See `Add Module with Script`_.

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

Add a new folder ``mynewcustomer``  inside ``path/to/nice2/customer`` with the following file structure

  .. figure:: resources/basic-folder-structure-customer.png

Open the file ``mynewcustomer/pom.xml`` and add the following content.

.. literalinclude:: resources/customer-module-pom-1.xml
   :language: XML

Open the file ``mynewcustomer/module/module/pom.xml`` and add the following content.

.. literalinclude:: resources/customer-module-pom-2.xml
   :language: XML

Open the file ``mynewcustomer/module/module/hiveapp-mount.properties`` and add the following content.

.. literalinclude:: resources/customer-module-hiveapp-mount.properties

Open the file ``mynewcustomer/module/module/descriptor/hivemodules.xml`` and add the following content.

.. literalinclude:: resources/customer-module-hivemodule.xml
   :language: XML


Add Application Properties
^^^^^^^^^^^^^^^^^^^^^^^^^^

For each customer module an ``application.properties`` file exists. The content of this file are properties which configure
the customer installation. Create a file ``application.properties`` in ``path/to/nice2/customer/CUSTOMERNAME/etc`` and
add the following content.

.. literalinclude:: resources/customer-module-application.properties

For a full list of all application properties see :ref:`application-properties`.

Add HikariCP Properties
^^^^^^^^^^^^^^^^^^^^^^^

`HikariCP`_ the JDBC connection pool used in Nice2. In order to work correctly with the customer installation, some
database properties must be set. For each customer module a ``hikaricp.properties`` file exists. Create a file
``hikaricp.properties`` in ``path/to/nice2/customer/CUSTOMERNAME/etc`` and add the following content.

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

Add Java Source Folders
^^^^^^^^^^^^^^^^^^^^^^^

As soon as any Java code is needed (e.g. for listeners, actions, services, rest-resources, ...) a Java module has to
be added to the module. There are three different types of Java modules which can be added.

* api -> defines services which can be injected by other modules
* spi -> defines classes which other modules can use or extend.
* impl -> the implementation of the module specific Java code


Add a new folder (impl, api or spi) to |pathToImplFolder| and add the following folder structure.

.. figure:: resources/impl-folder-structure.png

Open the file |pathToImplPom| and add the following content.

.. literalinclude:: resources/customer-module-impl-pom.xml
   :language: XML

Now the impl module has to be added to the module pom. Open the file |pathToModulePom| and add the impl module to the
modules element.

.. code-block:: XML
   :emphasize-lines: 3

   <modules>
     <module>module</module>
     <module>impl</module>
   </modules>

Now the impl module also has to be added as dependency to the module pom. Open the file |pathToModuleModulePom| and add
the impl module as dependency.

.. code-block:: XML

   <dependencies>
     <dependency>
       <groupId>ch.tocco.nice2.customer.mynewcustomer</groupId>
       <artifactId>nice2-customer-mynewcustomer-impl</artifactId>
       <version>${project.version}</version>
       <type>jar</type>
       <scope>compile</scope>
     </dependency>
   </dependencies>

Now Java files can be added in the folder ``java``.

.. include:: build-resources-into-target-snapshot.rst

.. include:: add-module-with-script.rst

.. _HikariCP: https://github.com/brettwooldridge/HikariCP
