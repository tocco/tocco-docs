New Persistence API
===================

In addition to the current persistence API (based on the :java:ref:`Context<ch.tocco.nice2.persist.Context>` class)
there is a new API that offers features that are not available in the old API.

.. warning::

    The new API is still evolving and features may be added or removed frequently.

PersistenceService
------------------

The :java:ref:`PersistenceService<ch.tocco.nice2.persist.hibernate.PersistenceService>` is the core of the new API
and provides type safe access to the persistence layer.
In contrast to the :java:ref:`Context<ch.tocco.nice2.persist.Context>` of the old API that only works with entity names,
the :java:ref:`PersistenceService<ch.tocco.nice2.persist.hibernate.PersistenceService>` also accepts entity classes and
returns an entity instance of the given class.
This cannot be used directly yet, because all entity classes are generated at startup and therefore cannot be referenced
by the code. However this functionality is currently used by the :java:ref:`QueryExecutor<ch.tocco.nice2.scripting.groovy.variable.QueryExecutor>`
that is available in groovy scripted listeners to provide statically compiled, type safe access to the entities.

In addition the :java:ref:`PersistenceService<ch.tocco.nice2.persist.hibernate.PersistenceService>` provides access
to the different query builder implementations. In addition to the standard query builder that returns :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>`
instances, there are other query builders that can select sub-paths directly and efficiently.

See :ref:`query_builder` for more information about this topic.

Metamodel
---------

The :java:ref:`Metamodel<ch.tocco.nice2.persist.hibernate.metamodel.Metamodel>` provides additional information about the data model
that is not available in the :java:ref:`DataModel<ch.tocco.nice2.persist.model.DataModel>`.

Currently the only functionality is to get the ``sql type`` of an entity field. This is used by the
:java:ref:`SchemaModelValidatorTask<ch.tocco.nice2.persist.backend.jdbc.impl.validate.SchemaModelValidatorTask>`
which checks if the database schema is compatible with the JPA schema.

