Memory management
=================

The new persistence layer based on Hibernate behaves a bit differently when a lot of data is loaded.
All entities that are loaded in a specific :java:ref:`Context<ch.tocco.nice2.persist.Context>` will be
referenced by the :java:extdoc:`Session<org.hibernate.Session>` until it is closed, even after the transaction
is committed.
When a lot of data is loaded or persisted within one action, the action should be split up into multiple transactions
and contexts.

PartitionedTask
---------------

The easiest way to split a big task into multiple transactions is to use a :java:ref:`PartitionedTask<ch.tocco.nice2.persist.exec.PartitionedTask>`.
A partitioned task is an extension of :java:ref:`PersistTask<ch.tocco.nice2.persist.exec.PersistTask>` and can be passed
to the :java:ref:`CommandExecutor<ch.tocco.nice2.persist.exec.CommandExecutor>`.

The :java:ref:`PartitionedTask<ch.tocco.nice2.persist.exec.PartitionedTask>` requires an inner task, a subclass of :java:ref:`AbstractPartitionedPersistTask<ch.tocco.nice2.persist.exec.AbstractPartitionedPersistTask>`,
that contains the execution logic.

The :java:ref:`AbstractPartitionedPersistTask<ch.tocco.nice2.persist.exec.AbstractPartitionedPersistTask>` has the following functionality:

    * A method ``partition()`` that converts it into a :java:ref:`PartitionedTask<ch.tocco.nice2.persist.exec.PartitionedTask>`,
      which then can be passed to the :java:ref:`CommandExecutor<ch.tocco.nice2.persist.exec.CommandExecutor>` for memory safe
      execution.

    * Managing the 'task data': Task data are :java:ref:`Context<ch.tocco.nice2.persist.Context>` dependent objects that
      are required for every iteration (for example an :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>`). Because a new context is initialized after a certain number
      of iterations, these objects need to be reinitialized for each context. The task data object can be defined by overriding
      the ``createTaskData()`` method. During task execution it can be accessed using the ``getTaskData(Context)`` method.
      It is automatically cached for each context, and recreated when a new context is created.

In addition to the inner task, it is possible to specify input and output transformation functions.
Remember that context dependent objects (like entities) may not be used as input parameters, therefore
:java:ref:`EntityId<ch.tocco.nice2.persist.entity.EntityId>` or :java:ref:`PrimaryKey<ch.tocco.nice2.persist.entity.PrimaryKey>`
should be used instead.
To facilitate the conversion between these objects, a :java:extdoc:`BiFunction<java.util.function.BiFunction>` can be passed
to the partitioned task. This function is executed each time a new partition is started and converts the input parameters before they
are passed to the inner task.

This means that it's possible to write the inner task using an :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>`
argument as usual, but pass :java:ref:`EntityId<ch.tocco.nice2.persist.entity.EntityId>` or :java:ref:`PrimaryKey<ch.tocco.nice2.persist.entity.PrimaryKey>`
instances to the :java:ref:`PartitionedTask<ch.tocco.nice2.persist.exec.PartitionedTask>`.

There are several standard transformation functions available, for example ``PartitionedTask#loadEntities()``.
Similar functions are available for the output value, for example to convert an :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` to an :java:ref:`EntityId<ch.tocco.nice2.persist.entity.EntityId>`.

The final argument is the size of the transaction, that is, how many iterations should be completed before a new
context is created.

.. note::

    It is currently only possible to split up into multiple transactions. Hibernate would offer the possibility to
    ``flush()`` and then ``clear()`` the session, without committing the transaction, however this is not available in the Tocco API yet
    (it's not clear which listeners to invoke on a ``flush()``).

Internally the :java:ref:`PartitionedTask<ch.tocco.nice2.persist.exec.PartitionedTask>` simply splits the input arguments
into partitions of the given transaction size.
For each partition a new :java:ref:`Context<ch.tocco.nice2.persist.Context>` created. Then the input transformation function
is applied and the inner task is executed.
After that the output transformation is applied to the result and the context is closed.
