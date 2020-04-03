.. _Validators:

Validators
==========

Validators are executed before the commit and are used to ensure data quality. They are executed in java and may
throw error messages if something is not quite right.

.. warning::
   Custom java validators must not be used if the requirement can be satisfied using standard entity validators (See :ref:`Entity-Validation`)

If a validator is required it can be created by implementing extending the ``AbstractEntitiesValidator`` class and implementing the ``validate``
method. The java code evaluates all entities of one entity model that were changed in a given transaction (first parameter of the ``validate`` method). If the entity validates successfully, nothing should be done
in the validator. If it does not validate the ``validationResults.get(entity).setError`` can be used to cancel the transaction and show an error message for each faulty entity. The message
can contain variables that may be filled in as described in :doc:`textresources`.

Please find below an example that validates ``entity`` and throws an error if ``relTest_status`` is not null and ``mandatory_if_status_set`` is not filled in.

.. code-block:: java

   public class TestValidator extends AbstractEntitiesValidator {
       @Override
       public void validate(List<Entity> entities, Map<Entity, EntityValidationResult> validationResults) {
           entities.stream()
               .filter(entity -> entity.getRelatedEntityOrNull("relTest_status") != null
                   && Strings.isNullOrEmpty(entity.getString("mandatory_if_status_set"))
               .forEach(entity -> validationResults.get(entity).setError(new TextMessage("validation.TestValidator.error_message"));
       }
   }

.. tip::
   Entity validation runs in the **NULL** business unit.

EntitiesValidator vs. EntityValidator
-------------------------------------

To optimise entity validation while importing entities the interface was changed for single entity validation (``AbstractEntityValidator``) to the current multi
entity validation. If entities / data structures need to resolved this can and should be done once per transaction instead of once per entity. See for example the
validator below. It checks if a default account, payment condition and currency is configured. This can now be done once per business unit instead of once
per entity which reduces the number of queries while importing addresses significantly.

.. code-block:: java

   public class DebitorInformationValidator extends AbstractEntitiesValidator {

       private static final String ERROR_MESSAGE = "validation.DebitorInformationValidator.missingDefaultMsg";

       private final Context context;
       private final QueryBuilderFactory queryBuilderFactory;
       private final BusinessUnitManager businessUnitManager;

       public DebitorInformationValidator(Context context,
                                          QueryBuilderFactory queryBuilderFactory,
                                          BusinessUnitManager businessUnitManager) {
           this.context = context;
           this.queryBuilderFactory = queryBuilderFactory;
           this.businessUnitManager = businessUnitManager;
       }

       @Override
       public void validate(List<Entity> debitorInformationEntities, Map<Entity, EntityValidationResult> validationResults) {
           Multimap<String, Entity> groupedDebitorInfoEntities = groupByBusinessUnit(debitorInformationEntities);

           for (String buId : groupedDebitorInfoEntities.keySet()) {
               if (defaultAccountNotExists(buId) || defaultPaymentConditionNotExists(buId) || defaultCurrencyNotExists(buId)) {
                   groupedDebitorInfoEntities.get(buId).forEach(entity -> validationResults.get(entity).setError(new TextMessage(ERROR_MESSAGE)));
               }
           }
       }

       private Multimap<String, Entity> groupByBusinessUnit(List<Entity> debitorInformationEntities) {
           Multimap<String, Entity> result = ArrayListMultimap.create();
           debitorInformationEntities.forEach(entity -> result.put(businessUnitManager.getBusinessUnit(entity).getId(), entity));
           return result;
       }

       private boolean defaultAccountNotExists(String buId) {
           return defaultNotExists("Account", "default_summary", buId);
       }

       private boolean defaultPaymentConditionNotExists(String buId) {
           return defaultNotExists("Payment_condition", "default_payment_condition", buId);
       }

       private boolean defaultCurrencyNotExists(String buId) {
           return defaultNotExists("Currency", "default_currency", buId);
       }

       private boolean defaultNotExists(String entityName, String defaultBooleanFieldName, String buId) {
           QueryBuilder queryBuilder = queryBuilderFactory.find(entityName)
                   .where(field(defaultBooleanFieldName).is(true));
           if (hasBuRelation(entityName)) {
               queryBuilder.where(field("relBusiness_unit.unique_id").is(buId));
           }
           EntityList defaultEntities = queryBuilder.build(context).execute();
           return defaultEntities.isEmpty();
       }

       private boolean hasBuRelation(String entityName) {
           return ((NiceEntityModel) context.getEntityManager(entityName).getModel()).getBusinessUnitType().isLinked();
       }
   }

All old validators will still work with the old interface. To make this posisble, a default implementation of the new interface
method was added to the old interface. **Please do not try to get rid of the old interface by copy-pasting the default implementation
to each validator.**

.. code-block:: java

   default void validate(List<Entity> entities, Map<Entity, EntityValidationResult> validationResults) {
       entities.forEach(entity -> validate(entity, validationResults.get(entity)));
   }

Registration
------------

Validators must be registered as services and contributed to the ``nice2.model.entity.EntityValidators`` contribution.

.. code-block:: xml

   <service-point id="TestValidator" interface="ch.tocco.nice2.model.entity.entityvalidators.EntitiesValidator">
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
