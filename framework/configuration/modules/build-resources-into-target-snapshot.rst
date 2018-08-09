Build Resources into Target Snapshot
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All resources must be built into the target snapshot. Otherwise the module would not work if it is deployed somewhere.
Resources are files with the following endings ``*.xml``, ``*.acl``, ``*.properties``, ``*.js``, ``*.ftl``.

To build static files into the target snapshot open the file |pathToModuleModulePom|.

Add all resource types you have added in your module folder by appending them to the ``build`` element. Take a look at
the following example:

.. code-block:: XML

   <build>
     <resources>
       <resource>
         <directory>descriptor</directory>
         <includes>
           <include>hivemodule.xml</include>
         </includes>
         <targetPath>.</targetPath>
       </resource>
       <resource>
         <directory>model</directory>
         <includes>
           <include>**/*.xml</include>
           <include>**/*.properties</include>
         </includes>
         <targetPath>model</targetPath>
       </resource>
       <resource>
         <directory>acl</directory>
         <includes>
           <include>*.acl</include>
         </includes>
         <targetPath>acl</targetPath>
       </resource>
       <resource>
         <directory>db</directory>
         <includes>
           <include>**/*.xml</include>
         </includes>
         <targetPath>db</targetPath>
       </resource>
    </resources>
  </build>

* create a ``resource`` element for each folder with resources
* inside the ``directory`` element the folder which contains any resources must be set.
* inside the ``include`` element it is specified what kind of files from this folder are built to the target snapshot.
