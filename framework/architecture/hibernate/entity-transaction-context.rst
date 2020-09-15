.. _transaction-context:

Transaction context
===================

The :nice:`EntityTransactionContext <ch/tocco/nice2/persist/hibernate/cascade/EntityTransactionContext>` collects
entities that have been created or deleted during the transaction and applies these changes to the session before
the transaction is committed.

All new entity instances are tracked automatically and the user of the API does not have to call ``Session#persist()``
(or equivalent) manually to add an entity to the persistence context.

All calls to ``Entity#delete()`` are recorded and then executed before the transaction is committed. The calls
will be reordered so that no constraint violations can occur on the database. That means it does not matter in which
order the entities are deleted by the user.

The service is a singleton and all changes are stored in maps using the current :java-hibernate:`Session <org/hibernate/Session>`
as key. The values are removed by the :nice:`TransactionControl <ch/tocco/nice2/persist/hibernate/TransactionControl>`
after each commit or rollback.

Created entities
----------------

In a first step all entities that are created and deleted in the same transaction are removed from the list
of entities to be inserted. This is necessary sometimes, as all entities are going to be saved automatically
when they are created within a transaction.

After that all newly created entities are passed to the :nice:`EntityInsertActionResolver <ch/tocco/nice2/persist/hibernate/cascade/EntityInsertActionResolver>`
which creates a list of :nice:`EntityInsertAction <ch/tocco/nice2/persist/hibernate/cascade/EntityInsertAction>`.
The returned list is ordered such that no constraint violations can occur when the insert statements are executed.

All that needs to be done is to make sure that unsaved entities which have non-nullable many-to-one reference
to another unsaved entity are saved *after* the referenced entity. All other cases (many-to-many associations or nullable
references) are correctly ordered by hibernate.

.. note::

    Also have a look at the javadoc in :abbr:`EntityTransactionContextTest (ch.tocco.nice2.persist.hibernate.EntityTransactionContextTest)`
    and :nice:`EntityInsertActionResolver <ch/tocco/nice2/persist/hibernate/cascade/EntityInsertActionResolver>`
    for more implementation details.

Entity factory
^^^^^^^^^^^^^^

All :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` instances are instantiated by the :nice:`EntityFactoryImpl <ch/tocco/nice2/persist/hibernate/pojo/EntityFactoryImpl>`:

    * All entities created by the user are delegated from ``PersistenceService#create()`` to the entity factory.
    * Entities that are instantiated by Hibernate (for example as a result of a query) are also delegated to
      the entity factory by the :nice:`CustomEntityPersister <ch/tocco/nice2/persist/hibernate/CustomEntityPersister>` (see :ref:`persister-entity-instantiation`).

The first purpose is to inject several necessary services into the :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>`
instances, for example the current :nice:`Context <ch/tocco/nice2/persist/Context>` or the :abbr:`NiceEntityModel (ch.tocco.nice2.model.NiceEntityModel)`.

In addition the following listeners are invoked when an entity was instantiated:

    * ``EntityFacadeListener#entityCreating`` is called for every newly created entity (this is *not* called for entities that are loaded from the database).
    * :nice:`EntityCreationListener <ch/tocco/nice2/persist/hibernate/EntityCreationListener>` are called for every
      created entity instance (``entityCreated()`` for new entities and ``entityLoaded()`` for existing entities).

Newly created entities are added to :nice:`EntityTransactionContext <ch/tocco/nice2/persist/hibernate/cascade/EntityTransactionContext>`
so that they will be persisted at the end of the transaction.

.. note::

    There is a caveat for entities loaded from the database: It is possible that the entity instantiation is actually
    the initialization of a :java-hibernate:`HibernateProxy <org/hibernate/proxy/HibernateProxy>`. In this case it is important
    to pass the proxy instance to the listeners (instead of the actual entity instance). Otherwise there are multiple
    entity instances representing the same database row, which will lead to unexpected side effects.

:nice:`EntityCreationListener <ch/tocco/nice2/persist/hibernate/EntityCreationListener>`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This listener (comparable to ``EntityFacadeListener#entityCreating``) is meant to be used by framework
code and will be called before all :nice:`EntityFacadeListener <ch/tocco/nice2/persist/entity/events/EntityFacadeListener>`
which are supposed to be used by business code.


Deleted entities
----------------

Similar considerations need to be made when deleting multiple entities. Entities that are being referenced by other
deleted entities must be deleted first to avoid constraint violation errors (which is the reverse order of the insert).

The deletion is done by the :abbr:`EntityDeletionUtils (ch.tocco.nice2.model.util.EntityDeletionUtils)` based on a
list of entity models.

A dependency map is created based on the existing relations between the entity models (a ``Multimap<NiceEntityModel, NiceEntityModel>``).
All entities of the key entity models must be deleted after the entities of the value entity models.

The following principles apply:

    * The entities on the many-to-one side of a bi-directional association need to be deleted first.
    * The owning side of a many-to-many should be deleted first (because the owning side manages the join table).
    * If the models depend on each other (many-to-one association from both sides) the side which has the non-nullable foreign key needs to be deleted first.

.. note::

    The inserting and deleting code cannot use the same ordering logic. See comments in the issue TOCDEV-312 for more details.

For performance reasons all deleted entities (either through ``Entity#delete()`` or through the delete query builder)
are collected and then deleted using a single statement.
Based on the ordering explained above, the following is executed per model:

The entities are deleted using a :java-javax:`CriteriaDelete <javax/persistence/criteria/CriteriaDelete>` query.

Because this deletes the entities directly from the database, we need to remove the deleted entities from the session manually.
First the entities are removed from loaded collections in the session (see ``DeleteEntityHelper#removeFromLoadedCollections()``)
and then the entities itself are detached from the session.

And finally the after commit event must be manually triggered as well (see ``AfterCommitListener#registerEntityDeletedEvent()``).

Before any delete query is executed, the session must be flushed to make sure that all ``UPDATE`` statements are executed first
(as they might reference an entity that will be deleted in the same transaction and because the :java-javax:`CriteriaDelete <javax/persistence/criteria/CriteriaDelete>`
queries are executed immediately and not when the session is flushed).

.. _delete_event_listener:

Removal of deleted entities from associations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To ensure that references to deleted entities are properly removed from many-to-many mapping tables they need to be
removed from the collection of the owning side of the association.
That means that the deleted entity has to be removed from every many-to-many collection where the owning side is not
the deleted entity.

This is done by the :abbr:`CustomDeleteEventListener (ch.tocco.nice2.persist.hibernate.cascade.CustomDeleteEventListener)`
which is a subclass of Hibernate's default :java-hibernate:`DefaultDeleteEventListener <org/hibernate/event/internal/DefaultDeleteEventListener>`.

.. note::

    It is important to carefully select the owning side as it might lead to performance problems if the collection
    of the owning side is very large (because it needs to be loaded to remove the deleted entity).

Before an entity is deleted, all nullable references to this entity will be set to ``NULL``. See :ref:`persister-delete`
for details. This also applies for batch deletions.