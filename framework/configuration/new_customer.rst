.. _add_customer_module:

Add new Customer module
=======================

 1. Go into your nice git repository
 2. Create the new customer module.

  .. code-block:: bash

    mkdir customer/${CUSTOMER}
    cd customer/${CUSTOMER}
    ../../src/bin/mkappmodule.sh

 3. Generate the pom dependencies. In backoffice execute "Modul"->"Aktionen"->"Modulabhängigkeiten"->"POM generieren"

  .. figure:: new_customer/modules.png
        :scale: 60%

 4. Check the generated dependencies, there are probably some mistakes in there. Add the dependencies to the customers pom

S3 module
^^^^^^^^^

Add this entry to the customer pom **if the customer should use S3**:

  .. code-block:: xml

    <dependency>
      <groupId>ch.tocco.nice2.optional.s3storage</groupId>
      <artifactId>nice2-optional-s3storage-module</artifactId>
      <version>${project.parent.version}</version>
      <type>appmodule</type>
      <scope>compile</scope>
    </dependency>
