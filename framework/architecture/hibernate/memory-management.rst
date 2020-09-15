Memory management
=================

The new persistence layer based on Hibernate behaves a bit differently when a lot of data is loaded.
All entities that are loaded in a specific :nice:`Context <ch/tocco/nice2/persist/Context>` will be
referenced by the :java-hibernate:`Session <org/hibernate/Session>` until it is closed, even after the transaction
is committed.
When a lot of data is loaded or persisted within one action, the action should be split up into multiple transactions
and contexts.

PartitionedTask
---------------

The easiest way to split a big task into multiple transactions is to use a :nice:`PartitionedTask <ch/tocco/nice2/persist/exec/PartitionedTask>`.
A partitioned task is an extension of :nice:`PersistTask <ch/tocco/nice2/persist/exec/PersistTask>` and can be passed
to the :nice:`CommandExecutor <ch/tocco/nice2/persist/exec/CommandExecutor>`.

The :nice:`PartitionedTask <ch/tocco/nice2/persist/exec/PartitionedTask>` requires an inner task, a subclass of :nice:`AbstractPartitionedPersistTask <ch/tocco/nice2/persist/exec/AbstractPartitionedPersistTask>`,
that contains the execution logic.

The :nice:`AbstractPartitionedPersistTask <ch/tocco/nice2/persist/exec/AbstractPartitionedPersistTask>` has the following functionality:

    * A method ``partition()`` that converts it into a :nice:`PartitionedTask <ch/tocco/nice2/persist/exec/PartitionedTask>`,
      which then can be passed to the :nice:`CommandExecutor <ch/tocco/nice2/persist/exec/CommandExecutor>` for memory safe
      execution.

    * Managing the 'task data': Task data are :nice:`Context <ch/tocco/nice2/persist/Context>` dependent objects that
      are required for every iteration (for example an :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>`). Because a new context is initialized after a certain number
      of iterations, these objects need to be reinitialized for each context. The task data object can be defined by overriding
      the ``createTaskData()`` method. During task execution it can be accessed using the ``getTaskData(Context)`` method.
      It is automatically cached for each context, and recreated when a new context is created.

In addition to the inner task, it is possible to specify input and output transformation functions.
Remember that context dependent objects (like entities) may not be used as input parameters, therefore
:nice:`EntityId <ch/tocco/nice2/persist/entity/EntityId>` or :nice:`PrimaryKey <ch/tocco/nice2/persist/entity/PrimaryKey>`
should be used instead.
To facilitate the conversion between these objects, a :java:`BiFunction <java/util/function/BiFunction>` can be passed
to the partitioned task. This function is executed each time a new partition is started and converts the input parameters before they
are passed to the inner task.

This means that it's possible to write the inner task using an :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>`
argument as usual, but pass :nice:`EntityId <ch/tocco/nice2/persist/entity/EntityId>` or :nice:`PrimaryKey <ch/tocco/nice2/persist/entity/PrimaryKey>`
instances to the :nice:`PartitionedTask <ch/tocco/nice2/persist/exec/PartitionedTask>`.

There are several standard transformation functions available, for example ``PartitionedTask#loadEntities()``.
Similar functions are available for the output value, for example to convert an :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` to an :nice:`EntityId <ch/tocco/nice2/persist/entity/EntityId>`.

The final argument is the size of the transaction, that is, how many iterations should be completed before a new
context is created.

.. note::

    It is currently only possible to split up into multiple transactions. Hibernate would offer the possibility to
    ``flush()`` and then ``clear()`` the session, without committing the transaction, however this is not available in the Tocco API yet
    (it's not clear which listeners to invoke on a ``flush()``).

Internally the :nice:`PartitionedTask <ch/tocco/nice2/persist/exec/PartitionedTask>` simply splits the input arguments
into partitions of the given transaction size.
For each partition a new :nice:`Context <ch/tocco/nice2/persist/Context>` created. Then the input transformation function
is applied and the inner task is executed.
After that the output transformation is applied to the result and the context is closed.

EntityList
----------

The behaviour of the different :nice:`EntityList <ch/tocco/nice2/persist/entity/EntityList>` implementations is a
bit different compared to the old persistence layer.

EntityListImpl
^^^^^^^^^^^^^^

The :nice:`EntityListImpl <ch/tocco/nice2/persist/hibernate/pojo/EntityListImpl>` is the default implementation.
It is based on a :java:`List <java/util/List>` of :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` instances.
These entities are already loaded, that means this implementation should not be used for very large lists, otherwise a lot of
memory will be required.

The :nice:`EntityListImpl <ch/tocco/nice2/persist/hibernate/pojo/EntityListImpl>` is mainly used as a result of the
``execute()`` method of the :nice:`Query <ch/tocco/nice2/persist/query/Query>` class.

.. note::

    Queries that are expected to have a lot of result rows should not use the ``execute()`` method. Instead ``getKeys()``
    or the :nice:`PathQueryBuilder <ch/tocco/nice2/persist/hibernate/query/builder/PathQueryBuilder>` should be used
    (perhaps in combination with a :nice:`PartitionedTask <ch/tocco/nice2/persist/exec/PartitionedTask>`).

LazyEntityList
^^^^^^^^^^^^^^

The :nice:`LazyEntityList <ch/tocco/nice2/persist/hibernate/LazyEntityList>` is based on a :nice:`PrimaryKeyList <ch/tocco/nice2/persist/entity/PrimaryKeyList>`.
No entities are loaded unless required and ``getKeys()`` can be called without any additional queries.

When an :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` is accessed, a number (see ``setPageSize()``) of entities
is loaded together.

This implementation works well, when only ``getKeys()`` (or only a few entities) are accessed. Also, it does not unnecessarily
load all entities, even when they are never used later.

However the loaded entities are always referenced by the list (and context) and high memory usage is still possible when
the entire list is loaded.

The :nice:`LazyEntityList <ch/tocco/nice2/persist/hibernate/LazyEntityList>` is returned from ``EntityManager#createEntityList(PrimaryKey...)``
and ``PrimaryKeyList#toEntityList()``.

MemoryEfficientLazyEntityList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :nice:`MemoryEfficientLazyEntityList <ch/tocco/nice2/persist/hibernate/MemoryEfficientLazyEntityList>` is also based on a
:nice:`PrimaryKeyList <ch/tocco/nice2/persist/entity/PrimaryKeyList>` and is based on pages like the
:nice:`LazyEntityList <ch/tocco/nice2/persist/hibernate/LazyEntityList>`.

The difference is that in the :nice:`MemoryEfficientLazyEntityList <ch/tocco/nice2/persist/hibernate/MemoryEfficientLazyEntityList>`
only one page is loaded at the same time. Each page is loaded with a new :nice:`Context <ch/tocco/nice2/persist/Context>`,
the previous :nice:`Context <ch/tocco/nice2/persist/Context>` is closed as soon as a new page is loaded.

This implementation implements the :java:`AutoCloseable <java/lang/AutoCloseable>` interface and should be used with the
try-with-resources pattern so that the last :nice:`Context <ch/tocco/nice2/persist/Context>` is closed properly.

This list can be used with very large sizes, because the memory of the previous page is freed when a new page is loaded
(or ``close()`` is called on the list).

.. warning::

    This list is only efficient when its elements are accessed in the given order. If the elements are accessed randomly,
    too many data is loaded from the database.

    :nice:`Entity <ch/tocco/nice2/persist/entity/Entity>` instances obtained from this list should only be used within
    the loop and primarily for read-only operations. As soon as its :nice:`Context <ch/tocco/nice2/persist/Context>`
    is closed, it's no longer possible to participate in a transaction or to load associations.

PrimaryKeyList
^^^^^^^^^^^^^^

The :nice:`PrimaryKeyList <ch/tocco/nice2/persist/entity/PrimaryKeyList>` is basically a ``List<PrimaryKey>``
with the following additional methods:

    * ``getModel()`` returns the corresponding :nice:`EntityModel <ch/tocco/nice2/persist/model/EntityModel>`
    * ``toEntityList()`` returns a :nice:`LazyEntityList <ch/tocco/nice2/persist/hibernate/LazyEntityList>` based on the keys of the list

It should be used where it can be expected that the size of the list is potentially very large, to indicate to the developer
that it's probably not a good idea to load all entities at once.