Collections
===========

To many relations are mapped by collections. We use a :java:extdoc:`LinkedHashSet<java.util.LinkedHashSet>` because
we want a unique collection that does preserve the order (to be able to define a default sorting).

In order to provide the same behaviour for collections as in the old API, some extensions were necessary.

Collection filtering
--------------------

All collection will be lazily initialized, this happens when a to many relation is resolved. However, we don't just want
to return all rows in the database. The elements in the collection should be filtered depending on, for example, the
current business unit or security.

`Hibernate Filters <https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#mapping-column-filter>`_
are unfortunately not flexible enough for our needs.
Therefore a custom :java:extdoc:`InitializeCollectionEventListener<org.hibernate.event.spi.InitializeCollectionEventListener>` was
implemented: :java:ref:`ExtendedInitializeCollectionEventListener<ch.tocco.nice2.persist.hibernate.interceptor.ExtendedInitializeCollectionEventListener>`.

Unfortunately it is not possible to simply provide a list of entities to initialize a collection, because they
are initialized directly from a :java:extdoc:`ResultSet<java.sql.ResultSet>` (see ``PersistentCollection#readFrom()``).
This is done when a :java:extdoc:`Loader<org.hibernate.loader.Loader>` contains a collection persister.
We did not find an appropriate way to implement such a loader to load collections with dynamically generated conditions,
therefore a workaround was necessary in :java:ref:`ExtendedInitializeCollectionEventListener<ch.tocco.nice2.persist.hibernate.interceptor.ExtendedInitializeCollectionEventListener>`:

.. code-block:: java

    ResultSet fake = createFakeResultSet();

    session.getPersistenceContext()
        .getLoadContexts()
        .getCollectionLoadContext(fake)
        .getLoadingCollection(loadedPersister, loadedKey);

    ...

    initializePersistentSet(collectionElements, (PersistentSet) collection);

    ...

    session.getPersistenceContext()
        .getLoadContexts()
        .getCollectionLoadContext(fake)
        .endLoadingCollections(loadedPersister);

First a fake :java:extdoc:`ResultSet<java.sql.ResultSet>` is created using the :java:extdoc:`ProxyFactory<javassist.util.proxy.ProxyFactory>`
(required as key) and then the persistence context is informed that a collection is about to be loaded.
Instead of using a :java:extdoc:`ResultSet<java.sql.ResultSet>` to initialize the collection, the collection elements
are added by reflection. After all elements are set, the persistence context is informed that collection loading is
completed.

The loading of the collection elements is delegated to an instance of :java:ref:`CollectionInitializer<ch.tocco.nice2.persist.hibernate.interceptor.CollectionInitializer>`.
Other modules can contribute collection initializer instances for specific relations to override the default behaviour.

The default :java:ref:`CollectionInitializer<ch.tocco.nice2.persist.hibernate.interceptor.CollectionInitializer>` is the
:java:ref:`DefaultCollectionInitializer<ch.tocco.nice2.persist.hibernate.interceptor.DefaultCollectionInitializer>`.
It creates the same query as Hibernate would, using the reverse association of the loaded collection.
However, because the query is executed through the :java:ref:`CriteriaQueryBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder>`,
the query is dynamically modified by all :java:ref:`QueryBuilderInterceptor<ch.tocco.nice2.persist.hibernate.query.QueryBuilderInterceptor>`, which
add conditions depending on the current user roles and business units.

.. todo::
    Add link to query builder chapter.

There are special implementations for entity-docs collections for performance reasons.

.. todo::
    Add link to entity-docs chapter.

.. warning::
    An empty collection does not mean that the corresponding database table is empty as well (all rows might be filtered).
    Hibernate tries to use some shortcuts if a collection is empty (for example clearing a many to many mapping table).
    To avoid this, we need to indicate to Hibernate, that the collection might not be empty:

    * ``ExtendedBasicCollectionPersister#isAffectedByEnabledFilters``
    * ``ReloadablePersistentSet#isSnapshotEmpty``


Collection reloading
--------------------

Per default a collection cannot be reloaded from the database once it has been initialized.
However, when a relation is resolved the collection should always be reloaded from the database because
a relation may be resolved multiple times within the same transaction with different privileges.

To support this, we use a custom persistent collection type, the :java:ref:`ReloadablePersistentCollectionType<ch.tocco.nice2.persist.hibernate.usertype.ReloadablePersistentCollectionType>`.
This type is configured for all collections (see :doc:`entity-class-generation`).

The concrete collection implementation is the :java:ref:`ReloadablePersistentSet<ch.tocco.nice2.persist.hibernate.usertype.ReloadablePersistentSet>`,
which has the following features:

Reloading
^^^^^^^^^

See ``ReloadablePersistentSet#reloadCollection``.

A collection can only be reloaded if it is already initialized and not transient.
If a collection is reloaded, all uncommitted changes will be lost, therefore we need to track them
so that they can be applied again after the reload.
These tracked changes must be reset after the session is flushed, this is done by overriding
``PersistentCollection#postAction()``.

.. warning::
    There is one case which is not supported:
    If an element is removed and the collection does no longer contain the removed element after the reload,
    an exception will be thrown, as the remove operation would be lost.

The code snippets which unload and then load the collection have been taken from different classes
of the Hibernate source code.

* The initialized flag of the collection needs to be reset to false (using reflection)
* The collection needs to be evicted from the session (based on code from :java:extdoc:`EvictVisitor<org.hibernate.event.internal.EvictVisitor>`)
* The collection needs to be loaded from the database and attached to the session again
* Uncommitted changes must be applied again

Delayed operation
^^^^^^^^^^^^^^^^^

Hibernate supports delayed (queued) operations, that get executed only after the collection was initialized.
This enables adding and removing elements without initializing the collection.
Queued operations cannot be used on the owning side of a many to many association because the owning side is
responsible for persisting the association. This means that the element has to be normally added to the collection
so that the change will be detected.
We use the delayed operations wherever possible (that means if an element is added to or removed from an uninitialized,
inverse collection) for performance reasons.
The queued operations are executed during ``PersistentCollection#afterInitialize()``. As our collection loading process
is different than normal, we call this manually from ``ReloadablePersistentSet#endRead()``.

.. note::
    An alternative to the implemented approach would be to use the standard collection handling and just run a
    query whenever a relation is resolved. It would then be required to synchronize the changes to the owning side of the
    association (this would cause an unnecessary collection load for many to many relations, unless the mapping table
    is mapped to an entity (which might make sense performance wise anyway)). This might a viable option in case the
    current approach fails with future hibernate versions.