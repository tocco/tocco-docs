Session Lifecycle
=================

The Hibernate :java-hibernate:`Session <org/hibernate/Session>` is the equivalent of the :abbr:`Context (ch.tocco.nice2.persist.Context)`
of the nice2 Persistence API.
The implementation (:abbr:`ContextAdapter (ch.tocco.nice2.persist.hibernate.legacy.ContextAdapter)`) of the Context
interface references exactly one session.

ContextService
--------------

The :abbr:`ContextService (ch.tocco.nice2.persist.hibernate.legacy.ContextService)` is a service that implements
the :abbr:`Context (ch.tocco.nice2.persist.Context)` interface.
This is the implementation that is injected into other services, when they need to access the :abbr:`Context (ch.tocco.nice2.persist.Context)`.
It is important to properly close a session to avoid memory leaks.

Implicitly and explicitly created contexts
------------------------------------------

There is exactly one implicit context per thread, which is created when the current context is accessed for
the first time (typically through the :abbr:`ContextService (ch.tocco.nice2.persist.hibernate.legacy.ContextService)`).
Unless the user manually creates additional contexts, all the persistence calls are handled by the implicit context.
It is stored in a :java:`ThreadLocal <java/lang/ThreadLocal>` variable in the :abbr:`ContextManagerAdapter (ch.tocco.nice2.persist.hibernate.legacy.ContextManagerAdapter)`.
The implicit context is automatically closed when the underlying session will be closed by the :abbr:`SessionFactoryManager (ch.tocco.nice2.persist.hibernate.session.SessionFactoryManager)`.

Sometimes it is necessary to create an isolated context for specific work. New contexts can be created by the
:abbr:`ContextManager (ch.tocco.nice2.persist.ContextManager)`. Unlike the implicit context, all additional contexts
must be manually closed by the user.

The :abbr:`ContextManagerAdapter (ch.tocco.nice2.persist.hibernate.legacy.ContextManagerAdapter)` maintains a
map which keeps track of all explicitly created contexts. This is necessary to be able to find a specific context instance by the
session it references.

This also makes it possible to attach an open context to a new thread. This is not recommended (the session is not thread safe),
but required by some legacy code.

Whenever a new context is created, a Hibernate session is opened and passed to this context.

Current session and current context
-----------------------------------

Current session
^^^^^^^^^^^^^^^

If the current session is requested from the :java-hibernate:`SessionFactory <org/hibernate/SessionFactory>` the call is delegated
to an implementation of :java-hibernate:`CurrentSessionContext <org/hibernate/context/spi/CurrentSessionContext>`.
We configure the :java-hibernate:`ManagedSessionContext <org/hibernate/context/internal/ManagedSessionContext>` by
setting the property ``hibernate.current_session_context_class`` (see :abbr:`HibernatePropertiesProvider (ch.tocco.nice2.persist.hibernate.HibernatePropertiesProvider)`).

The ManagedSessionContext requires the Session to be set explicitly when the current session is requested, otherwise an exception will be thrown.
In contrast, the previously used :java-hibernate:`ThreadLocalSessionContext <org/hibernate/context/internal/ThreadLocalSessionContext>` creates a new session when none was set, but it's a 'protected'
session that always requires a transaction and is not compatible with our API.
Thus it's better to just throw an exception when no session was set explicitly (as this should never occur anyway).

SessionFactoryManager
~~~~~~~~~~~~~~~~~~~~~

The :abbr:`SessionFactoryManager (ch.tocco.nice2.persist.hibernate.session.SessionFactoryManager)` manages the hibernate sessions.
All access to hibernate sessions should be made through this class!
This central management of sessions makes sure that the old :abbr:`Context (ch.tocco.nice2.persist.Context)`
based API can be used in combination with the new :abbr:`PersistenceService (ch.tocco.nice2.persist.hibernate.PersistenceService)`.

For example, when a new implicit session is created because the PersistenceService API has been accessed, ``ContextManager#getThreadContext()``
realizes that the implicit session already exists (even though no implicit Context instance exists yet) and re-uses this session.

This class holds a thread local reference to the 'implicit' session. This is the session that is created automatically when the persistence
layer is accessed for the first time during a request and no session has been opened explicitly.

If the current session is requested (``getCurrentSession()``), the session bound to the ManagedSessionContext is returned.
If nothing is bound, the implicit session is returned (and created if necessary) and bound to the ManagedSessionContext.
For every implicit context a :abbr:`ThreadCleanupListener (org.apache.hivemind.service.ThreadCleanupListener)` is registered
that detaches and closes the implicit session at the end of the request.

It is also possible to explicitly create a new session (using ``createNewSession()``). Explicitly created sessions
are always bound to the ManagedSessionContext. Explicitly created sessions need to be closed manually!
A :java-hibernate:`BaseSessionEventListener <org/hibernate/BaseSessionEventListener>` is registered with the session
which detaches the closed session and re-attaches the previous session (if there was one).

A :abbr:`SessionFactoryManagerListener (ch.tocco.nice2.persist.hibernate.session.SessionFactoryManagerListener)` can be registered
with the SessionFactoryManager. It's ``sessionClosing()`` method is called for every session that is about to be closed.

Current context
^^^^^^^^^^^^^^^

The current :abbr:`Context (ch.tocco.nice2.persist.Context)` is always the context which references the current session.
``ContextManagerAdapter#getThreadContext()`` returns the current context:

    - The current session is retrieved from the :abbr:`SessionFactoryManager (ch.tocco.nice2.persist.hibernate.session.SessionFactoryManager)`
      (this might create a new implicit session)
    - Check if there is an explicitly created context belonging to this sessions and return it (explicitly created contexts are
      cached in a ``Map``)
    - Check if the current session is the implicit session. If yes, check if there already is an implicit context instance
      for this thread and return it. If not, create a new implicit context instance and store it in the ThreadLocal. A
      :java-hibernate:`BaseSessionEventListener <org/hibernate/BaseSessionEventListener>` is added to this session, to make sure
      that the ThreadLocal is cleared when the implicit session is closed.
    - If none of the above applies, it must be an explicitly opened session --> create a context instance for it

Setting the current context
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The current session is set (or removed) when ``ContextAdapter#suspend()`` or ``ContextAdapter#resume()`` is called.
The session of that context is then bound to or detached from the current thread using the ``attachSessionToThread()``
or ``detachSessionFromThread()`` methods of the :abbr:`SessionFactoryManager (ch.tocco.nice2.persist.hibernate.session.SessionFactoryManager)`.
``ContextAdapter#resume()`` is called by default when a new context is created.

Flush mode
----------

We use ``FlushMode.COMMIT`` so that all changes in the session are flushed to the database just before the transaction is
committed.

We currently cannot use ``FlushMode.AUTO`` (which flushes all changes before a query, to make sure the query will return
up-to-date results), because we depend on commit listeners being executed before the changes are flushed to the database.