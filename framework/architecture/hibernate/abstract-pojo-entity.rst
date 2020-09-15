Abstract entity base class
==========================

All (non-transient) entities inherit from :nice:`AbstractPojoEntity <ch/tocco/nice2/persist/hibernate/pojo/AbstractPojoEntity>`.
This class provides all functionality required by the :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` interface.


Primary Key
-----------

In hibernate the primary key is simply a property (typically a :java:`Long <java/lang/Long>`) annotated
with :java-javax:`Id <javax/persistence/Id>`. In the old API the primary key value was encapsulated behind the
:nice:`PrimaryKey <ch/tocco/nice2/persist/entity/PrimaryKey>` interface.

An instance of :nice:`PrimaryKey <ch/tocco/nice2/persist/entity/PrimaryKey>` is created when ``getKey()`` or ``requireKey()``
is called for the first name. The key is cached so that always the same instance is returned, which is expected by
some of the business code.

Accessing values
----------------

PropertyAccessorService
^^^^^^^^^^^^^^^^^^^^^^^

The :nice:`PropertyAccessorServiceImpl <ch/tocco/nice2/persist/hibernate/pojo/PropertyAccessorServiceImpl>` efficiently
reads and writes entity properties.

Different strategies are used depending on the property type. For all persistent properties, the calls are delegated
to Hibernate's :java-hibernate:`EntityPersister <org/hibernate/persister/entity/EntityPersister>`.

Transient properties are manually accessed using reflection.

.. warning::

    It is important that all accessors are cached for
    performance reasons.

Reading values
^^^^^^^^^^^^^^

All calls to the different ``Entity#getValue()`` methods are delegated to ``AbstractHibernateEntity#internalGetValue()``,
where the actual field is resolved and read using the :nice:`PropertyAccessorService <ch/tocco/nice2/persist/hibernate/pojo/PropertyAccessorService>`.

For backwards compatibility, the resulting value is passed to ``TypeManager#isolate()`` before it is returned
(which creates a copy of :nice:`Binary <ch/tocco/nice2/persist/entity/Binary>` instances).

It is also attempted to convert the value to the requested type.

Writing values
^^^^^^^^^^^^^^

All calls to the different ``Entity#setValue()`` methods are delegated to ``AbstractHibernateEntity#internalSetValue()``,
where the actual field is resolved and updated using the :nice:`PropertyAccessorService <ch/tocco/nice2/persist/hibernate/pojo/PropertyAccessorService>`.

At first the value is converted to the required target type (if this is not already the case and a suitable
:nice:`Converter <ch/tocco/nice2/types/spi/Converter>` exists).

The resulting value is then compared to the old value - if they are the same, the method silently returns.
After the value has been set, a ``EntityFacadeListener#entityChanging()`` event will be fired.

.. warning::
    Calling ``internalGetValue()``, ``internalSetValue()`` bypasses the
    entity interceptors (security, localization etc). Therefore passing the field name to ``EntityInterceptor#accessField()`` is
    normally required before calling these methods. It may be omitted for certain internal calls where the interceptors
    are not required.

When Hibernate internally reads or writes properties of an entity, the field is accessed directly and no
additional code is executed.

If no transaction is running when ``setValue()`` is called (or a relation is changed) an exception will be thrown,
because otherwise these changes would be silently lost.

Resolving relations
-------------------

An association in hibernate is simply an instance of the referenced type (or a collection if it's a to-many relation).
In the old API it was required to 'resolve' a relation ( ``Entity#resolve()`` ) to a :nice:`RelationQuery <ch/tocco/nice2/persist/query/RelationQuery>`.
This relation query can then be executed to get an instance of :nice:`Relation <ch/tocco/nice2/persist/entity/Relation>`.

To-One relations
^^^^^^^^^^^^^^^^

All to one associations are explicitly configured to be loaded lazily (JPA default is eager).

:nice:`ToOneRelationQueryAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/ToOneRelationQueryAdapter>` is the
implementation of :nice:`RelationQuery <ch/tocco/nice2/persist/query/RelationQuery>` used for to-one associations.
It does not contain any special logic, it simply delegates the calls to the wrapped entity.

:nice:`ToOneRelationAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/ToOneRelationAdapter>` is the implementation
of :nice:`Relation <ch/tocco/nice2/persist/entity/Relation>` for to-one associations. This class implements getting, setting
and removing the associated instance.

All access (read or write) goes through the :nice:`RelationInterceptor <ch/tocco/nice2/persist/hibernate/RelationInterceptor>`,
this allows other modules to add functionality (for example security checks).
In order to enforce cleaner code, methods that were meant for to-many associations (for example ``RelationInterceptor#addEntity()``)
are not supported.

The :nice:`ToOneRelationAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/ToOneRelationAdapter>` provides the last
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

Collections are loaded lazily by default. We use a special implementation of the :java-hibernate:`PersistentSet <org/hibernate/collection/internal/PersistentSet>`
that supports reloading a collection from the database.

See :ref:`collection_reloading` for further information.

Every time a to-many relation is resolved, it should be reloaded from the database (because this is the behaviour of the
old persistence implementation).

:nice:`ToManyRelationQueryAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/ToManyRelationQueryAdapter>` is the
implementation of :nice:`RelationQuery <ch/tocco/nice2/persist/query/RelationQuery>` used for to-many associations.
It mainly delegates to the wrapped collection of entities.
However hibernate does not support pagination or (dynamic) sorting of associations, therefore these cases have to be
implemented specifically: If a relations needs to be resolved with a specific ordering or pagination an additional query
will be executed to get the desired results (the collection won't be touched). The results are returned as an
unmodifiable collection, because changes to this collection would be ignored (as it is unknown to hibernate).

:nice:`ToManyRelationAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/ToManyRelationAdapter>` is the implementation
of :nice:`Relation <ch/tocco/nice2/persist/entity/Relation>` for to-many associations. This class implements applying
modifications to the underlying collection.

Like its 'to-one' counterpart it implements the final :nice:`RelationInterceptor <ch/tocco/nice2/persist/hibernate/RelationInterceptor>`
that actually accesses the underlying collection and also enforces the usage of the correct methods.

If an operation (``addEntity`` or ``removeEntity``) causes a change:

* The underlying collection is modified
* The inverse association (in this case one-to-many or a many-to-many) is adjusted
* The ``EntityFacadeListener#entityRelationChanging()`` event is fired.

    * An event is fired if an entity has been added or removed (the ``adjusting`` flag is always false as there is only one event)
    * If all values are replaced using ``setEntities()``, first an event is fired for all removed entities. After that an event is fired for
      all newly added entities. If an entity is part of the collection before and after the operation, no add or
      remove event should be fired for this entity. The ``adjusting`` is always false, except for the very last event.

.. note::

    ``size()`` does not initialize the collection, but executes a ``COUNT`` query. This is important if the collection is
    large. However this means that ``size()`` should not be called when the collection is going to be initialized anyway
    (for example when ``iterator()`` or ``toList()`` is called), because that would lead to an unnecessary query.

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

See :ref:`delayed_operations` for further information about queued operations.

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
      instead it is checked whether an :java-hibernate:`EntityEntry <org/hibernate/engine/spi/EntityEntry>` for this entity
      exists.
    * Additionally if an entity has its primary key already set and its :java-hibernate:`EntityEntry <org/hibernate/engine/spi/EntityEntry>`
      status is ``SAVING`` the entity is also in conception state. This can happen when ``Entity#getState()`` is called from
      inside a validator (see ``ValidationInterceptor#onSave()``).

* ``INVALID``

    * If there is no :java-hibernate:`EntityEntry <org/hibernate/engine/spi/EntityEntry>` for an entity and it is not in
      conception state or deleted, it must be invalid.

* ``DIRTY``

    * See `Dirty checking`_

* ``CLEAN``

    * If all other states do not apply, the entity must be clean (that means persisted and unchanged).

Dirty checking
--------------

The :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` interface differentiates between ``touched`` and ``changed``
properties. A field is touched when ``setValue()`` has been called at least once for that field, even if the value is still the same.
As this distinction rarely makes sense, we no longer support it - only ``changed`` fields are returned from the
dirty checking methods (for example ``Entity#getChangedFields()`` or ``Entity#getTouchedFields()``).

The dirty fields are managed by the abstract base class :nice:`AbstractDirtyCheckingEntity <ch/tocco/nice2/persist/hibernate/pojo/AbstractDirtyCheckingEntity>`
in the ``changedFields`` property.
All calls to the setter methods are intercepted using a custom :nice:`PropertyAccessorService <ch/tocco/nice2/persist/hibernate/pojo/PropertyAccessorService>`.
If the value to be set is different from the `Old value`_, the field is marked as changed.

To check for modified collections (to-many relations) we can simply use the ``isDirty()`` method of the
:java-hibernate:`PersistentCollection <org/hibernate/collection/spi/PersistentCollection>`.

The list of changed fields needs to be reset when the changes are flushed to the database. This is done by the
:nice:`ValidationInterceptor <ch/tocco/nice2/persist/hibernate/validation/ValidationInterceptor>` after the entity
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

The :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` interface allows to query for the old value. This is the value
of a certain property when it was loaded from the database at the beginning of the transaction, ignoring all
uncommitted changes.

This is achieved by checking the 'loaded state' of the :java-hibernate:`EntityEntry <org/hibernate/engine/spi/EntityEntry>`,
which can be retrieved from the :java-hibernate:`PersistenceContext <org/hibernate/engine/spi/PersistenceContext>`.
This is where hibernate stores the state of the entity when it is loaded and this state is also used for hibernate's
default dirty checking mechanism.

EntityInterceptor
-----------------

The :nice:`EntityInterceptor <ch/tocco/nice2/persist/hibernate/EntityInterceptor>` interface allows
customizing the core entity functionality. The following functions can be intercepted:

    * Reading and writing fields
    * Deleting entities
    * Modifying relations

An entity interceptor instance is injected into every entity by the :nice:`EntityFactoryImpl <ch/tocco/nice2/persist/hibernate/pojo/EntityFactoryImpl>`.
The instance is created by the :nice:`EntityInterceptorFactoryImpl <ch/tocco/nice2/persist/hibernate/interceptor/EntityInterceptorFactoryImpl>`
which combines all interceptor contributions into an interceptor chain.
The inner most interceptor (which actually accesses the entity fields and so on) is provided by the entity itself
(``AbstractHibernateEntity#getInnerInterceptor()``).

.. note::

    The inner interceptor is wrapped in a :abbr:`LazyInterceptor (ch.tocco.nice2.persist.hibernate.interceptor.EntityInterceptorFactoryImpl.LazyInterceptor)`
    to avoid recursive proxy initialization (``Entity#getValue()`` -> ``proxy initialization`` ->
    ``EntityFactory#createInstance()`` -> ``Entity#getInnerInterceptor()`` -> ``proxy initialization`` ...).


Accessing values
^^^^^^^^^^^^^^^^

The method ``EntityInterceptor#accessField()`` can be used to intercept read or write access to a field.
It is always called when a value is accessed by the entity (typically when ``Entity#get/setValue()`` is called).

The default inner interceptor simply resolves the field name using the :nice:`FieldResolver <ch/tocco/nice2/persist/hibernate/interceptor/FieldResolver>`.
If write access is requested it additionally checks if the field is not a primary key or other generated field.

The :nice:`SecurityEntityInterceptorContribution <ch/tocco/nice2/persist/security/hibernate/SecurityEntityInterceptorContribution>`
uses this method to check the read or write permission of the given field. If the given field is a localized field, the base field (``label``
instead of ``label_de``) is used to check permissions.

Deleting entities
^^^^^^^^^^^^^^^^^

``EntityInterceptor#deleteEntity()`` is called when an entity is deleted (``Entity#delete()``).
The inner interceptor fires an ``EntityFacadeListener#entityDeleting()`` event and (unless the entity is unsaved)
schedules the entity for deletion with the :nice:`EntityTransactionContext <ch/tocco/nice2/persist/hibernate/cascade/EntityTransactionContext>`.

In addition the :nice:`SecurityEntityInterceptorContribution <ch/tocco/nice2/persist/security/hibernate/SecurityEntityInterceptorContribution>`
checks if the ``delete`` permission is granted for the current user.

Modifying relations
^^^^^^^^^^^^^^^^^^^

A :nice:`RelationInterceptor <ch/tocco/nice2/persist/hibernate/RelationInterceptor>` can be obtained from
the entity interceptor using ``createRelationInterceptor()``. The relation interceptor can be used to intercept
:nice:`Relation <ch/tocco/nice2/persist/entity/Relation>` modifications.

The inner interceptors are provided by the :nice:`AbstractRelationAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/AbstractRelationAdapter>`
implementations. These update the relation value or collection and fire an ``EntityFacadeListener#entityRelationChanging()`` event.

In addition the :nice:`SecurityEntityInterceptorContribution <ch/tocco/nice2/persist/security/hibernate/SecurityEntityInterceptorContribution>`
checks if the current user is allowed to modify a relation.

The :abbr:`BusinessUnitEntityInterceptor (ch.tocco.nice2.businessunit.impl.intercept.BusinessUnitEntityInterceptorContribution.BusinessUnitEntityInterceptor)`
checks if the business unit of an entity may be manually changed by the user (only business unit types ``MANUAL_SET``
and ``NONE`` may be changed by the user).

FieldResolver
^^^^^^^^^^^^^

The :nice:`FieldResolverImpl <ch/tocco/nice2/persist/hibernate/interceptor/FieldResolverImpl>` resolves a property name
to the name of the corresponding entity field.
Usually the property name is equal to the entity field name, however there are two exceptions:

    * Localized fields: if the base field of a localized field is requested (e.g. ``label``) it is resolved to the
      field of the current locale (e.g. ``label_de``).
    * When java reserved words are used as a field name in the entity model, the field name needs to be adjusted
      (see ``PojoUtils.normalizeFieldName()``).

It is called whenever a field is accessed or referenced by name, for example when reading or writing fields or when compiling
queries.