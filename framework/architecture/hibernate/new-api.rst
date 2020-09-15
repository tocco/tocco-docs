New Persistence API
===================

In addition to the current persistence API (based on the :nice:`Context <ch/tocco/nice2/persist/Context>` class)
there is a new API that offers features that are not available in the old API.

.. warning::

    The new API is still evolving and features may be added or removed frequently.

PersistenceService
------------------

The :nice:`PersistenceService <ch/tocco/nice2/persist/hibernate/PersistenceService>` is the core of the new API
and provides type safe access to the persistence layer.
In contrast to the :nice:`Context <ch/tocco/nice2/persist/Context>` of the old API that only works with entity names,
the :nice:`PersistenceService <ch/tocco/nice2/persist/hibernate/PersistenceService>` also accepts entity classes and
returns an entity instance of the given class.
This cannot be used directly yet, because all entity classes are generated at startup and therefore cannot be referenced
by the code. However this functionality is currently used by the :nice:`QueryExecutor <ch/tocco/nice2/scripting/groovy/variable/QueryExecutor>`
that is available in groovy scripted listeners to provide statically compiled, type safe access to the entities.

In addition the :nice:`PersistenceService <ch/tocco/nice2/persist/hibernate/PersistenceService>` provides access
to the different query builder implementations. In addition to the standard query builder that returns :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>`
instances, there are other query builders that can select sub-paths directly and efficiently.

See :ref:`query_builder` for more information about this topic.

PrimaryKeyLoader
^^^^^^^^^^^^^^^^

The :nice:`PrimaryKeyLoader <ch/tocco/nice2/persist/hibernate/interceptor/PrimaryKeyLoader>` is used to load
an entity by primary key. It is used when ``PersistenceService#retrieve()`` or ``EntityManager#get()`` is called.

The default implementation :abbr:`DefaultPrimaryKeyLoader (ch.tocco.nice2.persist.hibernate.pojo.PersistenceServiceImpl.DefaultPrimaryKeyLoader)`
uses the query builder to load the entity. This makes sure that the security conditions are always applied when an entity is loaded.

We cannot use Hibernate's ``Session#get()`` directly (even is we use a custom event listener that adds security conditions),
because it caches the results for the duration of the session. So if an entity is loaded in privileged mode first, it will
always remain accessible even if the privileged mode has ended.

There is also a special implementation for entity-docs (:nice:`EntityDocsPrimaryKeyLoader <ch/tocco/nice2/dms/impl/entitydocs/interceptor/EntityDocsPrimaryKeyLoader>`).
They need to be treated separately, as there are no ACL rules for them.

Metamodel
---------

The :nice:`Metamodel <ch/tocco/nice2/persist/hibernate/metamodel/Metamodel>` provides additional information about the data model
that is not available in the :nice:`DataModel <ch/tocco/nice2/persist/model/DataModel>`.

Currently the only functionality is to get the ``sql type`` of an entity field. This is used by the
:nice:`SchemaModelValidatorTask <ch/tocco/nice2/persist/backend/jdbc/impl/validate/SchemaModelValidatorTask>`
which checks if the database schema is compatible with the JPA schema.

