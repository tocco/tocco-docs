Session Lifecycle
=================

The Hibernate :java:extdoc:`Session<org.hibernate.Session>` is the equivalent of the :java:ref:`Context<ch.tocco.nice2.persist.Context>`
of the nice2 Persistence API.
The implementation (:java:ref:`ContextAdapter<ch.tocco.nice2.persist.hibernate.legacy.ContextAdapter>`) of the Context
interface references exactly one session.

ContextService
--------------

The :java:ref:`ContextService<ch.tocco.nice2.persist.hibernate.legacy.ContextService>` is a service that implements
the :java:ref:`Context<ch.tocco.nice2.persist.Context>` interface.
This is the implementation that is injected into other services, when they need to access the :java:ref:`Context<ch.tocco.nice2.persist.Context>`.
It is important to properly close a session to avoid memory leaks.
The :java:ref:`ContextService<ch.tocco.nice2.persist.hibernate.legacy.ContextService>` is a threaded service and implements
hivemind's :java:ref:`Discardable<org.apache.hivemind.Discardable>` interface to be able to close the implicit context easily
(see below).

Implicitly and explicitly created contexts
------------------------------------------

There is exactly one implicit context per thread, which is created when the current context is accessed for
the first time (typically through the :java:ref:`ContextService<ch.tocco.nice2.persist.hibernate.legacy.ContextService>`).
Unless the user manually creates additional contexts, all the persistence calls are handled by the implicit context.
It is stored in a :java:extdoc:`ThreadLocal<java.lang.ThreadLocal>` variable in the :java:ref:`ContextManagerAdapter<ch.tocco.nice2.persist.hibernate.legacy.ContextManagerAdapter>`.
The implicit context is automatically closed in the ``threadDidDiscardService()`` method, which is called by
hivemind at the end of the thread.

Sometimes it is necessary to create an isolated context for specific work. New contexts can be created by the
:java:ref:`ContextManager<ch.tocco.nice2.persist.ContextManager>`. Unlike the implicit context, all additional contexts
must be manually closed by the user.

The :java:ref:`ContextManagerAdapter<ch.tocco.nice2.persist.hibernate.legacy.ContextManagerAdapter>` maintains a
map which keeps track of all explicitly created contexts. This is necessary to be able to find a specific context instance by the
session it references.

This also makes it possible to attach an open context to a new thread. This is not recommended (the session is not thread safe),
but required by some legacy code.

Whenever a new context is created, a Hibernate session is opened and passed to this context.

.. note::
   The session should always be opened manually. If the current session is retrieved from the SessionFactory
   without manually opening a session, a temporary session is created which is closed after the first transaction
   is committed.

Current session and current context
-----------------------------------

Current session
^^^^^^^^^^^^^^^

If the current session is requested from the :java:extdoc:`SessionFactory<org.hibernate.SessionFactory>` the call is delegated
to an implementation of :java:extdoc:`CurrentSessionContext<org.hibernate.context.spi.CurrentSessionContext>`.
We configure the :java:extdoc:`ThreadLocalSessionContext<org.hibernate.context.internal.ThreadLocalSessionContext>` by
setting the property ``hibernate.current_session_context_class`` (see :java:ref:`HibernatePropertiesProvider<ch.tocco.nice2.persist.hibernate.HibernatePropertiesProvider>`).

Current context
^^^^^^^^^^^^^^^

The current :java:ref:`Context<ch.tocco.nice2.persist.Context>` is always the context which references the current session.
To retrieve the current context (see ``ContextManagerAdapter#getThreadContext()``) the current session is queried from
the session factory and then the matching context is retrieved from the thread local variables.

Setting the current context
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The current session is set (or removed) when ``ContextAdapter#suspend()`` or ``ContextAdapter#resume()`` is called.
The session of that context is then set to the :java:extdoc:`ThreadLocalSessionContext<org.hibernate.context.internal.ThreadLocalSessionContext>`.
``ContextAdapter#resume()`` is called by default when a new context is created.

Flush mode
----------

We use ``FlushMode.COMMIT`` so that all changes in the session are flushed to the database just before the transaction is
committed.

We currently cannot use ``FlushMode.AUTO`` (which flushes all changes before a query, to make sure the query will return
up-to-date results), because we depend on commit listeners being executed before the changes are flushed to the database.