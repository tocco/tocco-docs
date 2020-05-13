Entity History Implementation
=============================

The entry point for the history implementation is the :abbr:`HistoryService (ch.tocco.nice2.persist.history.v2.HistoryService)`.
This class is a :abbr:`ContextCreationListener (ch.tocco.nice2.persist.ContextCreationListener)`, which means that
it will be notified if a new :abbr:`Context (ch.tocco.nice2.persist.Context)` is created.

In order for this to work properly it needs to be loaded eagerly (``hiveapp.EagerLoad`` contribution) so that the
service can register itself with the :abbr:`ContextManager (ch.tocco.nice2.persist.ContextManager)` when the application
is initialized.

Whenever a new :abbr:`Context (ch.tocco.nice2.persist.Context)` is created, several listeners will be
added to the new context (all these listeners are removed again when the context is closed):

HistoryEntityFacadeListener
---------------------------

This listener is an :abbr:`EntityFacadeListener (ch.tocco.nice2.persist.entity.events.EntityFacadeListener)`,
which will be notified about all entity changes during a transaction.

Every entity that is modified during the transaction is stored in the context attribute ``history-snapshots-entities``.
Before the entity is stored, it is checked whether it should be included in the history using the
:abbr:`HistoryConfiguration (ch.tocco.nice2.persist.history.v2.config.HistoryConfiguration)` (a particular entity model
might be excluded from the history or the history might be disabled entirely).

PrepareHistoryDataListener
--------------------------

This listeners is a :abbr:`TransactionAware (ch.tocco.nice2.persist.tx.TransactionAware)` that is registered by the
:abbr:`HistoryService (ch.tocco.nice2.persist.history.v2.HistoryService)` with every newly created transaction.

The ``prepareCommit`` method of the :abbr:`TransactionAware (ch.tocco.nice2.persist.tx.TransactionAware)` is called
after all other listeners have already ran, so we can be sure that all changes have been captured by the
HistoryEntityFacadeListener at this point.

All entities that have been saved in the ``history-snapshots-entities`` attribute will be converted
into a :abbr:`DetachedEntity (ch.tocco.nice2.persist.history.v2.DetachedEntity)`, which contains all necessary information
for the snapshot that will be saved to the database. This has to be done before the commit, as some information is no longer available after commit
(changed fields for example).

Which fields and relations will be included in the snapshot depends on the entity model and is defined by the
:abbr:`HistoryConfiguration (ch.tocco.nice2.persist.history.v2.config.HistoryConfiguration)`.

All the created :abbr:`DetachedEntity (ch.tocco.nice2.persist.history.v2.DetachedEntity)` are stored in the context attribute ``history-detached-entities``
(the attribute ``history-snapshots-entities`` will be removed at this point).

HistoryWriter
-------------

This :abbr:`CommitListener (ch.tocco.nice2.persist.util.CommitListener)` is executed after the transaction is committed
(to make sure that the history is only written when the transaction has been committed successfully).
All the :abbr:`DetachedEntity (ch.tocco.nice2.persist.history.v2.DetachedEntity)` instances are read from the context attribute ``history-detached-entities``
and converted to an XML format using the XStream library. The XML structure is defined in the :abbr:`DetachedEntityMarshaller (ch.tocco.nice2.persist.history.v2.DetachedEntityMarshaller)`
and is exactly the same as in the previous history implementation (in order to be compatible with older history entries).
The XML String is gzipped before it is saved to save storage space.

The compressed XML data (along with other data like the username and ip address) are passed to the
:abbr:`HistoryDataStore (ch.tocco.nice2.persist.history.store.HistoryDataStore)` where they are persisted
in a dedicated history postgresql database. This is done asynchronously in a separate thread for performance reasons.

HistoryConfiguration
--------------------

The :abbr:`HistoryConfiguration (ch.tocco.nice2.persist.history.v2.config.HistoryConfiguration)` contains all information
whether the history is enabled for a certain entity model and if yes, which fields and relations should be included.

The history can be globally disabled using the ``nice2.persist.history.enabled`` property.
In addition it can also be disabled for specific entity models using the ``IgnoredEntityModels`` contribution.
Obviously no history entries will be created for session-only entities.

Which fields and relations are included in the snapshot is controlled by the :abbr:`EntityHistoryConfiguration (ch.tocco.nice2.persist.history.v2.EntityHistoryConfiguration)`.
There are default implementations for standard (:abbr:`DefaultEntityHistoryConfig (ch.tocco.nice2.persist.history.v2.config.DefaultEntityHistoryConfig)`)
and lookup entities (:abbr:`LookupEntityHistoryConfig (ch.tocco.nice2.persist.history.v2.config.LookupEntityHistoryConfig)`).
The ``IgnoredEntityModels`` contribution mentioned above can also be used the further refine the default implementations
by removing certain fields and relations from the snapshot.

However it is also possible to completely customize the history snapshot with a custom implementation (see
:abbr:`PageEntityHistoryConfiguration (ch.tocco.nice2.optional.cms.impl.history.PageEntityHistoryConfiguration)` for example).



