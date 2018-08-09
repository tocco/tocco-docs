Add Java Source Folders
^^^^^^^^^^^^^^^^^^^^^^^

As soon as any Java code is needed (e.g. for listeners, actions, services, rest-resources, ...) a Java module has to
be added to the module. There are three different types of Java modules which can be added.

* api -> defines services which can be injected by other modules
* spi -> defines classes which other modules can use or extend.
* impl -> the implementation of the module specific Java code


Add a new folder (impl, api, spi) to |pathToModuleFolder| and add the following folder structure.

.. figure:: resources/impl-folder-structure.png

now the impl module has to be added to the module pom. Open the file |pathToModulePom| and add the impl module to the
modules element.

.. code-block:: XML
   :emphasize-lines: 3

     <modules>
       <module>module</module>
       <module>impl</module>
     </modules>

  Now the impl module also has to be added as dependency to the module pom. Open the file |pathToModuleModulePom| and
  add the impl module as dependency.

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
