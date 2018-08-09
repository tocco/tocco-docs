.. |pathToModuleFolder| replace:: ``nice2-project/optional/mynewmodule``
.. |pathToModuleModuleFolder| replace:: ``nice2-project/optional/mynewmodule/module``
.. |pathToModulePom| replace:: ``nice2-project/optional/mynewmodule/pom.xml``
.. |pathToModuleModulePom| replace:: ``nice2-project/optional/mynewmodule/module/pom.xml``

Add Optional Module
===================

.. hint::
   This section explains how a new module is added manually. There is also a `script`_ which can be used to add a new module.

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

Add a new folder ``mynewmodule``  inside ``nice2-project/optional`` with the following file structure

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

Each module must be registered in the root pom file. Open the file ``nice2-project/pom.xml`` and add your module to it.
The modules are ordered alphabetical and are separated by core and optional modules.

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

The test customer has all possible modules installed. So a newly created optional module must be added to it. Open the
file ``nice2-project/customer/test/pom.xml`` and add the new module at the right place as dependency to it. All modules
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

.. include:: add-java-source-folder.rst

.. include:: build-resources-into-target-snapshot.rst

.. include:: add-module-with-script.rst

.. _script: `Add Module with Script`_
