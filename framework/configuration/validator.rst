.. _Validators:

Validators
==========

Validators are executed before the commit and are used to ensure data quality. They are executed in java and may
throw error messages if something is not quite right.

.. warning::
   Custom java validators must not be used if the requirement can be satisfied using standard entity validators (See :ref:`Entity-Validation`)

If a validator is required it can be created by implementing extending the ``AbstractEntityValidator`` class and implementing the ``validate``
method. The java code evaluates a single entity (first parameter of the ``validate`` method). If the entity validates successfully, nothing should be done
in the validator. If it does not validate the ``validationResult.setError`` can be used to cancel the transaction and show the given error message. The message
can contain variables that may be filled in as described in :ref:`Text-Resources`.

Please find below an example that validates ``entity`` and throws an error if ``relTest_status`` is not null and ``mandatory_if_status_set`` is not filled in.

.. code-block:: java

   public class TestValidator extends AbstractEntityValidator {
       @Override
       public void validate(@NotNull Entity entity, @NotNull EntityValidationResult validationResult) throws PersistException {
           if (entity.getRelatedEntityOrNull("relTest_status") != null &&
                   Strings.isNullOrEmpty(entity.getString("mandatory_if_status_set"))) {
               validationResult.setError(new TextMessage("validation.TestValidator.error_message"));
           }
       }
   }

.. tip::
   Entity validation runs in the **NULL** business unit.

Registration
------------

Validators must be registered as services and contributed to the ``nice2.model.entity.EntityValidators`` contribution.

.. code-block:: xml

   <service-point id="TestValidator" interface="ch.tocco.nice2.model.entity.entityvalidators.EntityValidator">
     <invoke-factory>
       <construct class="ch.tocco.nice2.optional.test.impl.validator.TestValidator"/>
     </invoke-factory>
   </service-point>

   <contribution configuration-id="nice2.model.entity.EntityValidators">
     <validator validator="service:TestValidator" filter="User, Address"/>
   </contribution>

The ``filter`` attribute defines which entities should be validated by this validator. If it is applicable for multiple entity models,
the entity model names may be listed seperated by commas. If a validator should be applied to all entities ``filter="*"`` can be used.

Dependencies
------------

hivemodule.xml
^^^^^^^^^^^^^^

In the ``hivemodule.xml`` of the module containing the validator, two ``ClassLoader`` imports are required to run validators.

.. code-block:: xml

   <contribution configuration-id="hiveapp.ClassLoader">
     <import feature="ch.tocco.nice2.model.entity" version="*"/>
     <import feature="ch.tocco.nice2.persist" version="*"/>
     <import feature="ch.tocco.nice2.textresources" version="*"/>
     <import feature="ch.tocco.nice2.validate" version="*"/>
   </contribution>

pom.xml
^^^^^^^

In the impl ``pom.xml`` of the module containing the validator, two dependencies are required to compile validators.

.. code-block:: xml

   <dependency>
     <groupId>ch.tocco.nice2.model.entity</groupId>
     <artifactId>nice2-model-entity-api</artifactId>
     <version>${project.version}</version>
     <scope>provided</scope>
   </dependency>
   <dependency>
     <groupId>ch.tocco.nice2.persist.core</groupId>
     <artifactId>nice2-persist-core-api</artifactId>
     <version>${project.version}</version>
     <scope>provided</scope>
   </dependency>
   <dependency>
     <groupId>ch.tocco.nice2.textresources</groupId>
     <artifactId>nice2-textresources-api</artifactId>
     <version>${project.version}</version>
     <type>jar</type>
     <scope>provided</scope>
   </dependency>
   <dependency>
     <groupId>ch.tocco.nice2.validate</groupId>
     <artifactId>nice2-validate-api</artifactId>
     <version>${project.version}</version>
     <type>jar</type>
     <scope>provided</scope>
   </dependency>