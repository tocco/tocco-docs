.. _Listeners:

Listeners
=========

This chapter explains the different kinds of listeners that exist, how and when they are called
and how they can be registered.

EntityFacadeListener
--------------------

An :nice:`EntityFacadeListener <ch/tocco/nice2/persist/entity/events/EntityFacadeListener>` is called
immediately after certain entity operations (e.g. ``EntityManager#create()`` or ``Entity#setValue()``) and therefore
this listener is independent of any hibernate functionality.

Normally entity facade listeners are registered as hivemind services which then are injected into the
:nice:`EntityFacadeListenerManagerImpl <ch/tocco/nice2/persist/hibernate/listener/EntityFacadeListenerManagerImpl>`,
which manages all listeners and can also be used to fire events.

It is also possible to temporarily register an :nice:`EntityFacadeListener <ch/tocco/nice2/persist/entity/events/EntityFacadeListener>` for the current context
using ``Context#addEntityFacadeListener()``. These listeners will be called for every entity and will only be called
for events of the context they belong to.
Likewise it is possible to add a temporary listener using ``EntityManager#addEntityFacadeListener()`` which will only
be called for events of this entity manager.

When an event is fired, the :nice:`EntityFacadeListenerManagerImpl <ch/tocco/nice2/persist/hibernate/listener/EntityFacadeListenerManagerImpl>`
searches for all registered hivemind listeners (that accept events of the affected entity) as well as all listeners of the current context
and entity manager.

``entityCreating`` event
^^^^^^^^^^^^^^^^^^^^^^^^

This event is fired directly after a new entity instance is created (no fields are set at this point) by the
:nice:`EntityFactoryImpl <ch/tocco/nice2/persist/hibernate/pojo/EntityFactoryImpl>`. All entity instances
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
:nice:`ToOneRelationAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/ToOneRelationAdapter>` and the
:nice:`ToManyRelationAdapter <ch/tocco/nice2/persist/hibernate/pojo/relation/ToManyRelationAdapter>`.
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

An :nice:`EntityListener <ch/tocco/nice2/persist/entity/events/EntityListener>` is used to listen
to 'after-commit' events. This means that these listeners are only called after a transaction has been
successfully committed.

For this purpose we use hibernate's :java-hibernate:`PostCommitInsertEventListener <org/hibernate/event/spi/PostCommitInsertEventListener>`,
:java-hibernate:`PostCommitUpdateEventListener <org/hibernate/event/spi/PostCommitUpdateEventListener>` and
:java-hibernate:`PostCommitDeleteEventListener <org/hibernate/event/spi/PostCommitDeleteEventListener>`.

These listener interfaces are implemented by :nice:`AfterCommitListenerImpl <ch/tocco/nice2/persist/hibernate/listener/AfterCommitListenerImpl>`,
which delegates the hibernate events to the corresponding :nice:`EntityListener <ch/tocco/nice2/persist/entity/events/EntityListener>`.
This class is bound to the hibernate events ``POST_COMMIT_INSERT``, ``POST_COMMIT_UPDATE`` and ``POST_COMMIT_DELETE``
(see :nice:`HibernateCoreBootstrapContribution <ch/tocco/nice2/persist/hibernate/bootstrap/HibernateCoreBootstrapContribution>`).

Listeners can either be contributed as hivemind services or registered temporarily through the ``Context`` or ``EntityManager``
(same as the entity facade listener).

Hibernate does not fire a ``POST_COMMIT_UPDATE`` for an entity if the only change is in a :term:`collection <hibernate collection>` and this collection is not the owning side of the association.
For this special use case there is the :nice:`CustomFlushEntityEventListener <ch/tocco/nice2/persist/hibernate/listener/CustomFlushEntityEventListener>`.
This is class is bound to the hibernate events ``FLUSH_ENTITY`` and checks every entity in the persistence context whether
this event needs to be fired manually.
If no event would be fired by hibernate but the entity has a change in (the non-owning side of) a :term:`collection <hibernate collection>`, the listener
registers a :java-hibernate:`AfterTransactionCompletionProcess <org/hibernate/action/spi/AfterTransactionCompletionProcess>`
(the event should only be fired if the transaction was completed successfully),
which fires the missing event manually, with the :java-hibernate:`ActionQueue <org/hibernate/engine/spi/ActionQueue>`.

CommitListener
--------------

A :nice:`CommitListener <ch/tocco/nice2/persist/hibernate/listener/CommitListener>` listens to events that are fired
just before or after a transaction is committed. The commit listeners are managed by the :nice:`EntityFacadeListenerManagerImpl <ch/tocco/nice2/persist/hibernate/listener/EntityFacadeListenerManagerImpl>`.

Commit listeners can be registered for the current context by calling ``Context#addCommitListener()``, which in turn
registers the listener with the :nice:`EntityFacadeListenerManagerImpl <ch/tocco/nice2/persist/hibernate/listener/EntityFacadeListenerManagerImpl>`.

As the :nice:`EntityFacadeListenerManagerImpl <ch/tocco/nice2/persist/hibernate/listener/EntityFacadeListenerManagerImpl>` tracks
all commit listeners by session in a map, it is important that they will be removed properly.
To avoid memory leaks when the user forgets to remove a commit listener, a :java-hibernate:`SessionEventListener <org/hibernate/SessionEventListener>`,
which removes all commit listeners when the session ends, is registered once per session.

The events are fired by the :abbr:`TransactionControlImpl (ch.tocco.nice2.persist.hibernate.PersistenceServiceImpl.TransactionControlImpl)` (see :doc:`transaction-lifecycle`)
just before or after the database transaction is committed. ``CommitListener#onAfterCommit()`` is only called if the commit
was successful.

The method ``afterFlush()`` is called after the hibernate session is flushed, but before the transaction is committed.
This means that all data that has been modified during this transaction is already available on the database when these listeners
are called. The method returns a ``boolean`` which indicates if the current listener has changed data on the database.
If ``true`` is returned, the session is flushed again before the ``afterFlush()`` method of the next listener is called.

This functionality is usually used through the :nice:`CollectingAfterFlushEntityListener <ch.tocco.nice2.persist.util.CollectingAfterFlushEntityListener>`.
Since this is the last listener that will be called before a transaction is committed, they must be ordered carefully using the ``priority()``
method, to make sure that no entity events are missed, because they were triggered by a listener that is executed later.

TransactionListener
-------------------

A :abbr:`TransactionListener (ch.tocco.nice2.persist.hibernate.ch.tocco.nice2.persist.hibernate.TransactionListener)` is another
listener that gets notified by transaction events. But in contrast to the :nice:`CommitListener <ch/tocco/nice2/persist/hibernate/listener/CommitListener>`
it is meant to be used internally by the persistence framework only.
This is a replacement of the :nice:`TransactionAware <ch/tocco/nice2/persist/tx/TransactionAware>` of the old persistence
implementation.

    - ``TransactionListener#onTransactionStart()`` is called when a new transaction has been started
    - ``TransactionListener#onCommit()`` is called after ``CommitListener#onBeforeCommit()`` has already been called
      and can be used to clean up resources for example.
    - ``TransactionListener#onRollback()`` is called just before a transaction will be rolled back
    - ``TransactionListener#afterTransaction()`` is called after every transaction (whether successful or not), but before ``CommitListener#onAfterCommit()``

A :abbr:`TransactionListener (ch.tocco.nice2.persist.hibernate.ch.tocco.nice2.persist.hibernate.TransactionListener)` can be registered with
the :nice:`TransactionControl <ch/tocco/nice2/persist/hibernate/TransactionControl>` of a transaction.

In addition it can also be added through the :nice:`PersistenceService <ch/tocco/nice2/persist/hibernate/PersistenceService>`
(``addTransactionListener()`` method). Listeners registered in this way will be applied to all transactions of the current
session and are passed to the :nice:`TransactionControl <ch/tocco/nice2/persist/hibernate/TransactionControl>` when
a new transaction is started.

ContextListener
---------------

The :nice:`ContextListener <ch/tocco/nice2/persist/ContextListener>` is part of the legacy API and contains two methods:

    * ``transactionStarted()`` is called when a new transaction has been started
    * ``contextDestroying()`` is called when a context is being closed

It can be registered using the ``Context#addContextListener()`` method and will be wrapped in a
:abbr:`ContextListenerAdapter (ch.tocco.nice2.persist.hibernate.legacy.ContextAdapter.ContextListenerAdapter)`.
The adapter class implements both :abbr:`TransactionListener (ch.tocco.nice2.persist.hibernate.ch.tocco.nice2.persist.hibernate.TransactionListener)`
(to implement the ``transactionStarted()`` method)
and :nice:`SessionFactoryManagerListener <ch/tocco/nice2/persist/hibernate/session/SessionFactoryManagerListener>`
(to implement the ``contextDestroying()`` method) to make sure that events are properly fired through both the old and new API.

ContextCreationListener
-----------------------

The :nice:`ContextCreationListener <ch/tocco/nice2/persist/ContextCreationListener>` is also implemented using an adapter class
to delegate the events to the ``SessionFactoryManagerListener#sessionCreated()`` event.

