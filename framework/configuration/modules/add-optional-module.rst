.. |pathToModuleFolder| replace:: ``path/to/nice2/optional/mynewmodule``
.. |pathToModuleModuleFolder| replace:: ``path/to/nice2/optional/mynewmodule/module``
.. |pathToModulePom| replace:: ``path/to/nice2/optional/mynewmodule/pom.xml``
.. |pathToModuleModulePom| replace:: ``path/to/nice2/optional/mynewmodule/module/pom.xml``
.. |pathToImplFolder| replace:: ``path/to/nice2/optional/mynewmodule/impl``
.. |pathToImplPom| replace:: ``path/to/nice2/optional/mynewmodule/impl/pom.xml``

Add Optional Module
===================

.. hint::
   This chapter describes how to add a new optional module manually. There is also a script which can be used to
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
     - `Add new Module to the Root Pom`_
   * - 4
     - ✔
     - `Add new Module to the Test Customer`_
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

Assume the new module is called ``mynewmodule``.

Add a new folder ``mynewmodule``  inside ``path/to/nice2/optional`` with the following file structure

.. figure:: resources/basic-folder-structure-optional.png

Open the file ``mynewmodule/pom.xml`` and add the following content.

.. literalinclude:: resources/optional-module-pom-1.xml
   :language: XML

Open the file ``mynewmodule/module/pom.xml`` and add the following content.

.. literalinclude:: resources/optional-module-pom-2.xml
   :language: XML

Open the file ``mynewmodule/module/hiveapp-mount.properties`` and add the following content.

.. literalinclude:: resources/optional-module-hiveapp-mount.properties

Open the file ``mynewmodule/module/descriptor/hivemodules.xml`` and add the following content.

.. literalinclude:: resources/optional-module-hivemodule.xml
   :language: XML

Add new Module to the Root Pom
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each module must be registered in the root pom file. Open the file ``path/to/nice2/pom.xml`` and add your module to it.
The modules are ordered alphabetically and are separated by core and optional modules.

.. code-block:: XML
   :emphasize-lines: 6

     <modules>
       <!-- ... more modules -->
       <module>optional/membershiphierarchylicence</module>
       <module>optional/membershiporder</module>
       <module>optional/membershipsms</module>
       <module>optional/mynewmodule</module>
       <module>optional/netmobile</module>
       <module>optional/news</module>
       <module>optional/newsletter</module>
       <module>optional/newsletterrecipient</module>
       <!-- ... more modules -->
     </modules>

Add new Module to the Test Customer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The test customer has all available modules installed. So a newly created optional module must be added to it. Open the
file ``path/to/nice2/customer/test/pom.xml`` and add the new module at the right place as dependency to it. All modules
are added in alphabetical order.

.. code-block:: XML
   :emphasize-lines: 10, 11, 12, 13, 14, 15, 16

      <dependencies>
        <!-- ... more modules -->
        <dependency>
          <groupId>ch.tocco.nice2.optional.membershipsms</groupId>
          <artifactId>nice2-optional-membershipsms-module</artifactId>
          <version>${project.parent.version}</version>
          <type>appmodule</type>
          <scope>compile</scope>
        </dependency>
        <dependency>
          <groupId>ch.tocco.nice2.optional.mynewmodule</groupId>
          <artifactId>nice2-optional-mynewmodule-module</artifactId>
          <version>${project.parent.version}</version>
          <type>appmodule</type>
          <scope>compile</scope>
        </dependency>
        <dependency>
          <groupId>ch.tocco.nice2.optional.cms</groupId>
          <artifactId>nice2-optional-cms-module</artifactId>
          <version>${project.parent.version}</version>
          <type>appmodule</type>
          <scope>compile</scope>
        </dependency>
        <!-- ... more modules -->
      </dependencies>

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


Add a new folder (impl, api or spi) to |pathToImplFolder| and add the following folder structure. If an api module has to
be added, replace ``impl`` with ``api``.

.. figure:: resources/impl-folder-structure.png

Open the file |pathToImplPom| and add the following content.

.. literalinclude:: resources/optional-module-impl-pom.xml
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
       <groupId>ch.tocco.nice2.optional.mynewmodule</groupId>
       <artifactId>nice2-optional-mynewmodule-impl</artifactId>
       <version>1.0-SNAPSHOT</version>
       <type>jar</type>
       <scope>compile</scope>
     </dependency>
   </dependencies>

Now Java files can be added in the folder ``java``.

.. hint::
   The process would be the same for an ``api`` or ``spi`` module.


.. include:: build-resources-into-target-snapshot.rst

.. include:: add-module-with-script.rst
