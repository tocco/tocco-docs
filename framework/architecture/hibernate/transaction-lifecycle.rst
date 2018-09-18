Transaction Lifecycle
=====================

A session is associated with exactly one transaction at the same time. It is however possible to run
several transactions consecutively in the same session.
A transaction is only required for write operations.

TransactionControl
------------------

There are several ways to start a transaction:

* using the :java:ref:`TxInvoker<ch.tocco.nice2.persist.impl.entity.TxInvoker>` obtained from ``Context#tx()``
* using the :java:ref:`TransactionManager<ch.tocco.nice2.persist.tx.TransactionManager>` directly
* using the :java:ref:`PersistService<ch.tocco.nice2.persist.hibernate.PersistService>` of the new API

Independently of how the transaction was started, in the background an instance of :java:ref:`TransactionControl<ch.tocco.nice2.persist.hibernate.TransactionControl>`
will be created. No matter through which API the transaction is accessed, always the same transaction control instance
is used per transaction.

The transaction control has the following responsibilities:

* it encapsulates the actual Hibernate :java:extdoc:`Transaction<org.hibernate.Transaction>`
* commit or rollback of the transaction
* firing of commit listeners
* handling of created and deleted entities

Commit Listeners
----------------

CommitListener
^^^^^^^^^^^^^^

A :java:ref:`CommitListener<ch.tocco.nice2.persist.hibernate.listener.CommitListener>` can be registered with the
:java:ref:`EntityFacadeListenerManager<ch.tocco.nice2.persist.hibernate.listener.EntityFacadeListenerManager>` for
a specific session.

Commit listeners implemented using the interface of the old API (:java:ref:`CommitListener<ch.tocco.nice2.persist.util.CommitListener>`)
are registered by the :java:ref:`Context<ch.tocco.nice2.persist.Context>` using an adapter class.

These listeners are meant to be used by the business code to run actions before or after a commit (or rollback).

TransactionListener
^^^^^^^^^^^^^^^^^^^

The :java:ref:`TransactionListener<ch.tocco.nice2.persist.hibernate.TransactionListener>` can be registered
directly on the transaction control and is meant to be used by internal code and are used to clean up the transaction.

:java:ref:`TransactionAware<ch.tocco.nice2.persist.tx.TransactionAware>` instances which are registered on the
:java:ref:`TransactionAdapter<ch.tocco.nice2.persist.hibernate.legacy.TransactionAdapter>` are also added as
:java:ref:`TransactionListener<ch.tocco.nice2.persist.hibernate.TransactionListener>` using an adapter class.

Transaction context
-------------------

The :java:ref:`EntityTransactionContext<ch.tocco.nice2.persist.hibernate.cascade.EntityTransactionContext>` keeps
track of all entities that are created or deleted during the transaction and executes these changes before the
transaction is committed.

See :ref:`transaction-context` for more details.