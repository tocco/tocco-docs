.. _persister:

Custom persisters
=================

Custom entity persister
-----------------------

The :java:extdoc:`EntityPersister<org.hibernate.persister.entity.EntityPersister>` interface contains information how to
map an entity to the database table. There is one instance per mapped class.
We extend Hibernate's default implementations to achieve some custom behaviour (:java:ref:`CustomEntityPersister<ch.tocco.nice2.persist.hibernate.CustomEntityPersister>`).

Lazy initialization of UniqueEntityLoader cache
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Hibernate initializes an instance of :java:extdoc:`UniqueEntityLoader<org.hibernate.loader.entity.UniqueEntityLoader>`
per :java:extdoc:`LockMode<org.hibernate.LockMode>`. This can use quite a lot of memory if there are a lot of entity classes.
Since we almost exclusively use the standard lock mode a lot of these loaders will never be used. In addition, these loaders
are primarily used to load an entity by primary key, which we rarely use because we almost always use the query builder
(to make sure business unit and security conditions are added).

To avoid wasting memory for unnecessary loaders they are instantiated lazily when they are used for the first time.
A second reason for the lazy initialization is to optimize the startup performance, so that the application is
ready as soon as possible.

.. note::
    This can perhaps be removed when Hibernate is upgraded to 5.3+ (see HHH-12558).

.. _persister-delete:

Custom delete behaviour
^^^^^^^^^^^^^^^^^^^^^^^

Before an entity is deleted the persister sets all (nullable) references (which point to the deleted entity) to ``NULL``
to avoid constraint violations.
This was introduced in order to be compatible with existing code (as it is the default behaviour of the old persistence
framework).
For each 'one to many' association (whose inverse side is nullable) the following query is executed:
``UPDATE nice_entity SET relReverse = NULL WHERE relReverse = IN (:obj)`` (``:obj`` are the entities to be deleted).

.. note::
    It is important that these queries are executed directly before the delete statements are executed
    (instead of for example doing it in :java:extdoc:`DeleteEventListener<org.hibernate.event.spi.DeleteEventListener>`.)
    Otherwise the ``NULL`` values might be overridden by an update statement.

.. _persister-entity-instantiation:

Entity instantiation
^^^^^^^^^^^^^^^^^^^^

By default Hibernate instantiates entity classes by invoking their default constructor.
Entity instantiation is intercepted by overriding ``EntityPersister#instantiate()``; the instantiation itself is then
delegated to the :java:ref:`EntityFactory<ch.tocco.nice2.persist.hibernate.pojo.EntityFactory>`.
The :java:ref:`EntityFactory<ch.tocco.nice2.persist.hibernate.pojo.EntityFactory>` injects services into
the created entities, tracks new entities and invokes listeners.

Before the entity factory is called it is verified whether the entity to be created belongs to the current
Session/Context. This is important as otherwise the wrong Session/Context would be injected by the entity factory.

Custom collection persister
---------------------------

There is an instance of a :java:extdoc:`CollectionPersister<org.hibernate.persister.collection.CollectionPersister>` for
every collection. Like with the entity persister, a customized implementation is used.

Filtered collections
^^^^^^^^^^^^^^^^^^^^

By always returning true from ``isAffectedByEnabledFilters()`` Hibernate assumes that the collection might be filtered.
Even though we do not use Hibernate's filter feature we use a similar concept (see :ref:`collections`).
When filters are enabled certain shortcuts cannot be used by Hibernate (for example removing all entries in a n:n
mapping table, which might wrongfully remove filtered data from the database).

Lazy initialization of CollectionInitializer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:extdoc:`CollectionInitializer<org.hibernate.loader.collection.CollectionInitializer>` instances are also lazily
initialized for the same reasons as above (memory usage and startup performance).