Transaction Lifecycle
=====================

A session is associated with exactly one transaction at the same time. It is however possible to run
several transactions consecutively in the same session.
A transaction is only required for write operations.

TransactionControl
------------------

There are several ways to start a transaction:

* using the :abbr:`TxInvoker (ch.tocco.nice2.persist.impl.entity.TxInvoker)` obtained from ``Context#tx()``
* using the :nice:`TransactionManager <ch/tocco/nice2/persist/tx/TransactionManager>` directly
* using the :nice:`PersistenceService <ch/tocco/nice2/persist/hibernate/PersistenceService>` of the new API

Independently of how the transaction was started, in the background an instance of :nice:`TransactionControl <ch/tocco/nice2/persist/hibernate/TransactionControl>`
will be created. No matter through which API the transaction is accessed, always the same transaction control instance
is used per transaction.

The transaction control has the following responsibilities:

* it encapsulates the actual Hibernate :java-hibernate:`Transaction <org/hibernate/Transaction>`
* commit or rollback of the transaction
* firing of commit listeners
* handling of created and deleted entities

Commit Listeners
----------------

CommitListener
^^^^^^^^^^^^^^

A :nice:`CommitListener <ch/tocco/nice2/persist/hibernate/listener/CommitListener>` can be registered with the
:nice:`EntityFacadeListenerManager <ch/tocco/nice2/persist/hibernate/listener/EntityFacadeListenerManager>` for
a specific session.

Commit listeners implemented using the interface of the old API (:nice:`CommitListener <ch/tocco/nice2/persist/util/CommitListener>`)
are registered by the :nice:`Context <ch/tocco/nice2/persist/Context>` using an adapter class.

These listeners are meant to be used by the business code to run actions before or after a commit (or rollback).

TransactionListener
^^^^^^^^^^^^^^^^^^^

The :nice:`TransactionListener <ch/tocco/nice2/persist/hibernate/TransactionListener>` can be registered
directly on the transaction control and is meant to be used by internal code and are used to clean up the transaction.

:nice:`TransactionAware <ch/tocco/nice2/persist/tx/TransactionAware>` instances which are registered on the
:nice:`TransactionAdapter <ch/tocco/nice2/persist/hibernate/legacy/TransactionAdapter>` are also added as
:nice:`TransactionListener <ch/tocco/nice2/persist/hibernate/TransactionListener>` using an adapter class.

Transaction context
-------------------

The :nice:`EntityTransactionContext <ch/tocco/nice2/persist/hibernate/cascade/EntityTransactionContext>` keeps
track of all entities that are created or deleted during the transaction and executes these changes before the
transaction is committed.

See :ref:`transaction-context` for more details.

Validation
----------

Entities that have been created or modified during a transaction will be validated before the transaction is committed.

The validation for new entities is started in ``EntityTransactionContext#executeEntityOperations()`` just before
``Session#save()`` is called. ``ValidationContext.Operation.INSERT`` is passed to the validation context for newly created entities.

The validation of updated entities is started by the :nice:`ValidationInterceptor <ch/tocco/nice2/persist/hibernate/validation/ValidationInterceptor>`
(which is a Hibernate :java-hibernate:`Interceptor <org/hibernate/Interceptor>`).

All modified entities are validated by the ``preFlush()`` event that is called for all entities which are in the Hibernate session
before the changes are flushed to the database. Only dirty entities will be validated.
``ValidationContext.Operation.UPDATE`` is passed to the validation context for updated entities.
