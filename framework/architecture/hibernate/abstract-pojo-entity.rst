Abstract entity base class
==========================

All (non-transient) entities inherit from :java:ref:`AbstractPojoEntity<ch.tocco.nice2.persist.hibernate.pojo.AbstractPojoEntity>`.
This class provides all functionality required by the :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` interface.


Primary Key
-----------

In hibernate the primary key is simply a property (typically a :java:extdoc:`Long<java.lang.Long>`) annotated
with :java:extdoc:`Id<javax.persistence.Id>`. In the old API the primary key value was encapsulated behind the
:java:ref:`PrimaryKey<ch.tocco.nice2.persist.entity.PrimaryKey>` interface.

An instance of :java:ref:`PrimaryKey<ch.tocco.nice2.persist.entity.PrimaryKey>` is created when ``getKey()`` or ``requireKey()``
is called for the first name. The key is cached so that always the same instance is returned, which is expected by
some of the business code.

Dirty checking
--------------

The :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` interface differentiates between ``touched`` and ``changed``
properties. A field is touched when ``setValue()`` has been called at least once for that field, even if the value is still the same.
As this distinction rarely makes sense, we no longer support it - only ``changed`` fields are returned from the
dirty checking methods (for example ``Entity#getChangedFields()`` or ``Entity#getTouchedFields()``).

The dirty fields are managed by the abstract base class :java:ref:`AbstractDirtyCheckingEntity<ch.tocco.nice2.persist.hibernate.pojo.AbstractDirtyCheckingEntity>`.
All calls to the setter methods are intercepted. If the value to be set is different from the `Old value`_, the field
is marked as changed.
To check for modified collections (to-many relations) we can simply use the ``isDirty()`` method of the
:java:extdoc:`PersistentCollection<org.hibernate.collection.spi.PersistentCollection>`.

The list of changed fields needs to be reset when the changes are flushed to the database. This is done by the
:java:ref:`ValidationInterceptor<ch.tocco.nice2.persist.hibernate.validation.ValidationInterceptor>` after the entity
validation has been completed.

.. note::
    Instead of manually keeping track of all the changes it would be possible to just always compare the current value
    with the old value, when we need the changed fields. However this is a bit of a performance problem, because the
    changed fields are needed quite often, especially by ``Entity#getState()`` to check if the current state is ``DIRTY``.

.. todo::
    Perhaps `hibernate bytecode enhancement <https://docs.jboss.org/hibernate/orm/5.2/topical/html_single/bytecode/BytecodeEnhancement.html>`_
    may be used in the future.

Old value
---------

The :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` interface allows to query for the old value. This is the value
of a certain property when it was loaded from the database at the beginning of the transaction, ignoring all
uncommitted changes.

This is achieved by checking the 'loaded state' of the :java:extdoc:`EntityEntry<org.hibernate.engine.spi.EntityEntry>`,
which can be retrieved from the :java:extdoc:`PersistenceContext<org.hibernate.engine.spi.PersistenceContext>`.
This is where hibernate stores the state of the entity when it is loaded and this state is also used for hibernate's
default dirty checking mechanism.

Accessing values
----------------

Reading values
^^^^^^^^^^^^^^

All calls to the different ``Entity#getValue()`` methods are delegated to ``AbstractHibernateEntity#internalGetValue()``,
where the actual (generated) getter method is resolved and called by reflection.

For backwards compatibility, the resulting value is passed to ``TypeManager#isolate()`` before it is returned
(which creates a copy of :java:ref:`Binary<ch.tocco.nice2.persist.entity.Binary>` instances).

Writing values
^^^^^^^^^^^^^^

All calls to the different ``Entity#setValue()`` methods are delegated to ``AbstractHibernateEntity#internalSetValue()``,
where the actual (generated) setter method is resolved and called by reflection.

At first the value is converted to the required target type (if this is not already the case and a suitable
:java:ref:`Converter<ch.tocco.nice2.types.spi.Converter>` exists).

The resulting value is then compared to the old value - if they are the same, the method silently returns.
After the value has been set, a ``EntityFacadeListener#entityChanging()`` event will be fired.

.. warning::
    Calling ``internalGetValue()``, ``internalSetValue()`` or the generated getter and setter methods bypasses the
    entity interceptors (security, localization etc). Therefore passing the field name to ``EntityInterceptor#accessField()`` is
    normally required before calling these methods. It may be omitted for certain internal calls where the interceptors
    are not required.

When Hibernate internally reads or writes the properties of an entity, the generated getter and setter methods are
called directly.

Resolving relations
-------------------

An association in hibernate is simply an instance of the referenced type (or a collection if it's a to-many relation).
In the old API it was required to 'resolve' a relation ( ``Entity#resolve()`` ) to a :java:ref:`RelationQuery<ch.tocco.nice2.persist.query.RelationQuery>`.
This relation query can then be executed to get an instance of :java:ref:`Relation<ch.tocco.nice2.persist.entity.Relation>`.

To-One relations
^^^^^^^^^^^^^^^^

All to one associations are explicitly configured to be loaded lazily (JPA default is eager).

:java:ref:`ToOneRelationQueryAdapter<ch.tocco.nice2.persist.hibernate.pojo.relation.ToOneRelationQueryAdapter>` is the
implementation of :java:ref:`RelationQuery<ch.tocco.nice2.persist.query.RelationQuery>` used for to-one associations.
It does not contain any special logic, it simply delegates the calls to the wrapped entity.

:java:ref:`ToOneRelationAdapter<ch.tocco.nice2.persist.hibernate.pojo.relation.ToOneRelationAdapter>` is the implementation
of :java:ref:`Relation<ch.tocco.nice2.persist.entity.Relation>` for to-one associations. This class implements getting, setting
and removing the associated instance.

All access (read or write) goes through the :java:ref:`RelationInterceptor<ch.tocco.nice2.persist.hibernate.RelationInterceptor>`,
this allows other modules to add functionality (for example security checks).
In order to enforce cleaner code, methods that were meant for to-many associations (for example ``RelationInterceptor#addEntity()``)
are not supported.

The :java:ref:`ToOneRelationAdapter<ch.tocco.nice2.persist.hibernate.pojo.relation.ToOneRelationAdapter>` provides the last
interceptor in the chain, which actually accesses the underlying entity.

* Reading the value means simply calling ``Entity#getValue()`` on the source entity. Internally this calls the generated
  getter for the association.
* When a value is written some more actions are performed (if the new value is the same as the current one, the call is silently ignored):

    * The updated value is set to the source entity using ``Entity#setValue()``
    * As all associations in nice2 are bi-directional, the inverse association (in this case always a one-to-many association)
      needs to be updated. The previous value (if present) needs to be removed, and the new
      value (if not null) needs to be added from/to the inverse association.
    * The ``EntityFacadeListener#entityRelationChanging()`` event is fired.

        * If the previous value was not null, an event is fired for removing the old value (the ``adjusting`` flag is true if the new value is not null)
        * If the new value is not null, an event is fired for adding the new value.

To-many relations
^^^^^^^^^^^^^^^^^

Collections are loaded lazily by default. We use a special implementation of the :java:extdoc:`PersistentSet<org.hibernate.collection.internal.PersistentSet>`
that supports reloading a collection from the database.

.. todo::
    Link to chapter that explains reloading collections

Every time a to-many relation is resolved, it should be reloaded from the database (because this is the behaviour of the
old persistence implementation).

:java:ref:`ToManyRelationQueryAdapter<ch.tocco.nice2.persist.hibernate.pojo.relation.ToManyRelationQueryAdapter>` is the
implementation of :java:ref:`RelationQuery<ch.tocco.nice2.persist.query.RelationQuery>` used for to-many associations.
It mainly delegates to the wrapped collection of entities.
However hibernate does not support pagination or (dynamic) sorting of associations, therefore these cases have to be
implemented specifically: If a relations needs to be resolved with a specific ordering or pagination an additional query
will be executed to get the desired results (the collection won't be touched). The results are returned as an
unmodifiable collection, because changes to this collection would be ignored (as it is unknown to hibernate).

:java:ref:`ToManyRelationAdapter<ch.tocco.nice2.persist.hibernate.pojo.relation.ToManyRelationAdapter>` is the implementation
of :java:ref:`Relation<ch.tocco.nice2.persist.entity.Relation>` for to-many associations. This class implements applying
modifications to the underlying collection.

Like its 'to-one' counterpart it implements the final :java:ref:`RelationInterceptor<ch.tocco.nice2.persist.hibernate.RelationInterceptor>`
that actually accesses the underlying collection and also enforces the usage of the correct methods.

If an operation (``addEntity`` or ``removeEntity``) causes a change:

* The underlying collection is modified
* The inverse association (in this case one-to-many or a many-to-many) is adjusted
* The ``EntityFacadeListener#entityRelationChanging()`` event is fired.

    * An event is fired if an entity has been added or removed (the ``adjusting`` flag is always false as there is only one event)
    * If all values are replaced using ``setEntities()``, first an event is fired for all removed entities. After that an event is fired for
      all newly added entities. If an entity is part of the collection before and after the operation, no add or
      remove event should be fired for this entity. The ``adjusting`` is always false, except for the very last event.

Syncing inverse associations
----------------------------

When the user changes an association, the other side should be updated automatically by the framework,
as all associations are bi-directional at the moment.

When doing this, care must be taken not to unnecessarily initialize lazy collections, as this would have a negative
performance impact.
On the other hand, sometimes this is necessary in many to many associations, when the user did not update the owning
side (see :doc:`entity-class-generation`).

* If the reverse side is a many-to-one association it can just be updated without any performance penalty (it is also necessary to do so
  because the many-to-one side is always the owning side).
* If the reverse side is a one-to-many or inverse many-to-many association, the collection may be updated but the
  addition may be queued if the collection is not initialized yet.
* If the reverse side is the owning side of many-to-many association, the collection must always be updated (and perhaps
  initialized). Otherwise the changes would not be persisted by hibernate.

.. todo::
    Link to chapter explaining the queued operations.

.. note::
    In the future it might be worth to check if we want to explicitly map the mapping table with an entity class. This would allow
    using many-to-one/one-to-many associations and avoid unnecessary collection initialization.

Entity state
------------

The states are checked in the following order (important):

* ``PHANTOM``

    * The phantom state is tracked by the ``wasDeleted`` field. This is necessary because of two reasons.
      First, the actual delete query is not immediately fired (but just before the transaction is committed,
      to make sure that all delete statements are executed in the correct order), but the state has to be PHANTOM
      immediately after the ``delete()`` method was called. Second, after the session is flushed, the deleted entity
      is no longer in Hibernate's persistence context, so it would not be possible to tell if an entity is deleted
      using ``EntityEntry#getStatus()``.

* ``CONCEPTION``

    * If an entity has a primary key which is auto-generated by the database and this key is null, the state of the
      entity must be conception. For primary keys which are generated by the user (for example strings) this does not work,
      instead it is checked whether an :java:extdoc:`EntityEntry<org.hibernate.engine.spi.EntityEntry>` for this entity
      exists.
    * Additionally if an entity has its primary key already set and its :java:extdoc:`EntityEntry<org.hibernate.engine.spi.EntityEntry>`
      status is ``SAVING`` the entity is also in conception state. This can happen when ``Entity#getState()`` is called from
      inside a validator (see ``ValidationInterceptor#onSave()``).

* ``INVALID``

    * If there is no :java:extdoc:`EntityEntry<org.hibernate.engine.spi.EntityEntry>` for an entity and it is not in
      conception state or deleted, it must be invalid.

* ``DIRTY``

    * See `Dirty checking`_

* ``CLEAN``

    * If all other states do not apply, the entity must be clean (that means persisted and unchanged).

EntityHolder
------------

The :java:ref:`EntityHolder<ch.tocco.nice2.persist.entity.EntityList.EntityHolder>` is an interface that
is returned from :java:ref:`EntityList<ch.tocco.nice2.persist.entity.EntityList>` or :java:ref:`Relation<ch.tocco.nice2.persist.entity.Relation>`
that wraps an entity. This interface is deprecated and should not be used anymore, but it is still necessary to support it,
as it is still referenced in a lot of existing code.
Normally an instance of an entity holder just delegates all calls to the wrapped entity. However Hibernate cannot work with these
wrapped entities, because they do not have the generated getters and setters.

The solution to this problem is that the :java:ref:`AbstractPojoEntity<ch.tocco.nice2.persist.hibernate.pojo.AbstractPojoEntity>`
also implements the :java:ref:`EntityHolder<ch.tocco.nice2.persist.entity.EntityList.EntityHolder>` interface. For this to work
correctly, the iterators of the entity lists had to be extended, so that they do not wrap an entity, if the entity
already implements the :java:ref:`EntityHolder<ch.tocco.nice2.persist.entity.EntityList.EntityHolder>` interface
(see :java:ref:`EntityHolderIteratorWrapper<ch.tocco.nice2.persist.entity.EntityHolderIteratorWrapper>`).