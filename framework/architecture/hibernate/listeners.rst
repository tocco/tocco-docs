.. _Listeners:

Listeners
=========

This chapter explains the different kinds of listeners that exist, how and when they are called
and how they can be registered.

EntityFacadeListener
--------------------

An :abbr:`EntityFacadeListener (ch.tocco.nice2.persist.entity.events.EntityFacadeListener)` is called
immediately after certain entity operations (e.g. ``EntityManager#create()`` or ``Entity#setValue()``) and therefore
this listener is independent of any hibernate functionality.

Normally entity facade listeners are registered as hivemind services which then are injected into the
:abbr:`EntityFacadeListenerManagerImpl (ch.tocco.nice2.persist.hibernate.listener.EntityFacadeListenerManagerImpl)`,
which manages all listeners and can also be used to fire events.

It is also possible to temporarily register an :abbr:`EntityFacadeListener (ch.tocco.nice2.persist.entity.events.EntityFacadeListener)` for the current context
using ``Context#addEntityFacadeListener()``. These listeners will be called for every entity and will only be called
for events of the context they belong to.
Likewise it is possible to add a temporary listener using ``EntityManager#addEntityFacadeListener()`` which will only
be called for events of this entity manager.

When an event is fired, the :abbr:`EntityFacadeListenerManagerImpl (ch.tocco.nice2.persist.hibernate.listener.EntityFacadeListenerManagerImpl)`
searches for all registered hivemind listeners (that accept events of the affected entity) as well as all listeners of the current context
and entity manager.

``entityCreating`` event
^^^^^^^^^^^^^^^^^^^^^^^^

This event is fired directly after a new entity instance is created (no fields are set at this point) by the
:abbr:`EntityFactoryImpl (ch.tocco.nice2.persist.hibernate.pojo.EntityFactoryImpl)`. All entity instances
are created by the entity factory (even those that are loaded from the database), but the event is only fired when a new instance
is created by the user (not when an entity is loaded from the database). This is the case when no primary key is passed
to the entity factory.

``entityDeleting`` event
^^^^^^^^^^^^^^^^^^^^^^^^

This event is fired when ``Entity#delete()`` is called.
Note that the entity won't be deleted from the database until the transaction is committed.

``entityChanging`` event
^^^^^^^^^^^^^^^^^^^^^^^^

This event is fired every time a field is updated (i.e. if ``Entity#setValue()`` or the setter for this field is called).
This is done in ``AbstractHibernateEntity#internalSetValue()`` which is called by both ``Entity#setValue()`` and the
generated setter method.

``entityRelationChanging`` event
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This event is fired every time a relation is modified (i.e. if an entity is added to or removed from a relation).
This is done in ``AbstractRelationAdapter#fireRelationChangingEvent()`` which in turn is called by the
:abbr:`ToOneRelationAdapter (ch.tocco.nice2.persist.hibernate.pojo.relation.ToOneRelationAdapter)` and the
:abbr:`ToManyRelationAdapter (ch.tocco.nice2.persist.hibernate.pojo.relation.ToManyRelationAdapter)`.
An event is also always fired for the reverse side of the relation.

See the chapter :doc:`abstract-pojo-entity` for a description when the ``adjusting`` flag is set to true.

``entityReceivedValues`` event (deprecated)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This event is fired when a newly created entity receives its primary key after it was inserted into the database.
See ``EntityTransactionContextImpl#executeEntityOperations``.
This event is rarely used and mostly implemented for compatibility reasons. New listeners should not use this event
anymore.

.. _flush_event:

EntityListener
--------------

An :abbr:`EntityListener (ch.tocco.nice2.persist.entity.events.EntityListener)` is used to listen
to 'after-commit' events. This means that these listeners are only called after a transaction has been
successfully committed.

For this purpose we use hibernate's :java-hibernate:`PostCommitInsertEventListener <org/hibernate/event/spi/PostCommitInsertEventListener>`,
:java-hibernate:`PostCommitUpdateEventListener <org/hibernate/event/spi/PostCommitUpdateEventListener>` and
:java-hibernate:`PostCommitDeleteEventListener <org/hibernate/event/spi/PostCommitDeleteEventListener>`.

These listener interfaces are implemented by :abbr:`AfterCommitListenerImpl (ch.tocco.nice2.persist.hibernate.listener.AfterCommitListenerImpl)`,
which delegates the hibernate events to the corresponding :abbr:`EntityListener (ch.tocco.nice2.persist.entity.events.EntityListener)`.
This class is bound to the hibernate events ``POST_COMMIT_INSERT``, ``POST_COMMIT_UPDATE`` and ``POST_COMMIT_DELETE``
(see :abbr:`HibernateCoreBootstrapContribution (ch.tocco.nice2.persist.hibernate.bootstrap.HibernateCoreBootstrapContribution)`).

Listeners can either be contributed as hivemind services or registered temporarily through the ``Context`` or ``EntityManager``
(same as the entity facade listener).

Hibernate does not fire a ``POST_COMMIT_UPDATE`` for an entity if the only change is in a :term:`collection <hibernate collection>` and this collection is not the owning side of the association.
For this special use case there is the :abbr:`CustomFlushEntityEventListener (ch.tocco.nice2.persist.hibernate.listener.CustomFlushEntityEventListener)`.
This is class is bound to the hibernate events ``FLUSH_ENTITY`` and checks every entity in the persistence context whether
this event needs to be fired manually.
If no event would be fired by hibernate but the entity has a change in (the non-owning side of) a :term:`collection <hibernate collection>`, the listener
registers a :java-hibernate:`AfterTransactionCompletionProcess <org/hibernate/action/spi/AfterTransactionCompletionProcess>`
(the event should only be fired if the transaction was completed successfully),
which fires the missing event manually, with the :java-hibernate:`ActionQueue <org/hibernate/engine/spi/ActionQueue>`.

CommitListener
--------------

A :abbr:`CommitListener (ch.tocco.nice2.persist.hibernate.listener.CommitListener)` listens to events that are fired
just before or after a transaction is committed. The commit listeners are managed by the :abbr:`EntityFacadeListenerManagerImpl (ch.tocco.nice2.persist.hibernate.listener.EntityFacadeListenerManagerImpl)`.

Commit listeners can be registered for the current context by calling ``Context#addCommitListener()``, which in turn
registers the listener with the :abbr:`EntityFacadeListenerManagerImpl (ch.tocco.nice2.persist.hibernate.listener.EntityFacadeListenerManagerImpl)`.

As the :abbr:`EntityFacadeListenerManagerImpl (ch.tocco.nice2.persist.hibernate.listener.EntityFacadeListenerManagerImpl)` tracks
all commit listeners by session in a map, it is important that they will be removed properly.
To avoid memory leaks when the user forgets to remove a commit listener, a :java-hibernate:`SessionEventListener <org/hibernate/SessionEventListener>`,
which removes all commit listeners when the session ends, is registered once per session.

The events are fired by the :abbr:`TransactionControlImpl (ch.tocco.nice2.persist.hibernate.PersistenceServiceImpl.TransactionControlImpl)` (see :doc:`transaction-lifecycle`)
just before or after the database transaction is committed. ``CommitListener#onAfterCommit()`` is only called if the commit
was successful.

TransactionListener
-------------------

A :abbr:`TransactionListener (ch.tocco.nice2.persist.hibernate.ch.tocco.nice2.persist.hibernate.TransactionListener)` is another
listener that gets notified by transaction events. But in contrast to the :abbr:`CommitListener (ch.tocco.nice2.persist.hibernate.listener.CommitListener)`
it is meant to be used internally by the persistence framework only.
This is a replacement of the :abbr:`TransactionAware (ch.tocco.nice2.persist.tx.TransactionAware)` of the old persistence
implementation.

    - ``TransactionListener#onCommit()`` is called after ``CommitListener#onBeforeCommit()`` has already been called
      and can be used to clean up resources for example.
    - ``TransactionListener#onRollback()`` is called just before a transaction will be rolled back
    - ``TransactionListener#afterTransaction()`` is called after every transaction (whether successful or not), but before ``CommitListener#onAfterCommit()``

A :abbr:`TransactionListener (ch.tocco.nice2.persist.hibernate.ch.tocco.nice2.persist.hibernate.TransactionListener)` can be registered with
the :abbr:`TransactionControl (ch.tocco.nice2.persist.hibernate.TransactionControl)` of a transaction.
