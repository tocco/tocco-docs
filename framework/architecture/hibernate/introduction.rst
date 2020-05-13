Introduction
============

Originally the Persistence API was a completely self-made framework.

This implementation has now been replaced by Hibernate (currently version 5.4.x), due to the following reasons:

* Lack of documentation and test coverage
* Lost knowledge (original developers no longer work here)
* Missing features (complex queries, typed entities)

In particular it was crucial to be able to execute queries that select (multiple) simple properties instead
of entire entities to improve the performance of multiple aspects of nice2.

This document describes...

* how the existing API has been implemented using Hibernate as backend (necessary to support existing code)
* the new API

This document does not describe the usage of the old Persistence API or Hibernate itself.

At first it is described how Hibernate is initialized (:doc:`session-factory-provider`) and how the
XML entity definitions are converted into Java classes (:doc:`entity-class-generation`).

Next it is explained how the automatically generated Java classes implement the functionality of the
:abbr:`Entity (ch.tocco.nice2.persist.entity.Entity)` interface (:doc:`abstract-pojo-entity`) and how
the :abbr:`Relation (ch.tocco.nice2.persist.entity.Relation)` interface is mapped to the JPA associations
(:doc:`collections`).

There is additional in depth information available about the mapping of the data:

    * :doc:`user-types`
    * :doc:`entity-persister`
    * :doc:`entity-tuplizer`
    * :doc:`generated-values`

The next chapters describes the lifecycle of the :abbr:`Context (ch.tocco.nice2.persist.Context)` (:doc:`session-lifecycle`)
and the :abbr:`Transaction (ch.tocco.nice2.persist.tx.Transaction)` (:doc:`transaction-lifecycle` and :doc:`entity-transaction-context`).

It is also explained how the different kind of listeners are integrated into the persistence layer (:doc:`listeners`).

Probably the most significant improvement is the new query builder (:doc:`query-builder`) that allows much more efficient queries than before.
The query builder is part of the new persistence API (:doc:`new-api`) that is still under development.

Finally there are chapter about a few specific topics:

    * How binary data is stored (:doc:`large-objects`)
    * How to handle large amounts of data (:doc:`memory-management`)
    * How to log SQL statements for debugging purposes (:doc:`sql-logging`)