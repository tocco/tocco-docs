.. _query_builder:

Query Builder
=============

There are currently 4 different query builders for different query types (all of which can be obtained from the
:java:ref:`PersistenceService<ch.tocco.nice2.persist.hibernate.PersistenceService>`):

    * The :java:ref:`EntityQueryBuilder<ch.tocco.nice2.persist.hibernate.query.EntityQueryBuilder>` builds queries that
      return :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` instances. It is convenient to use the :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>`
      interface, but it might have a performance impact when too much data is loaded or too many queries are executed.
    * The :java:ref:`SinglePathQueryBuilder<ch.tocco.nice2.persist.hibernate.query.SinglePathQueryBuilder>` can be used to
      fetch exactly one property or path of an entity (for example to get all primary keys of a given query). This avoids
      unnecessarily loading the entire :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>`.
    * The :java:ref:`PathQueryBuilder<ch.tocco.nice2.persist.hibernate.query.PathQueryBuilder>` is similar to the :java:ref:`SinglePathQueryBuilder<ch.tocco.nice2.persist.hibernate.query.SinglePathQueryBuilder>`
      but can select more than one path and always returns an ``Object[]`` as a result.

In addition there is the :java:ref:`SubqueryBuilder<ch.tocco.nice2.persist.hibernate.query.SubqueryBuilder>` which is used
to create sub-queries. An instance of this can be acquired from the :java:ref:`SubqueryFactory<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder.SubqueryFactory>`
of another query builder.

Internally, JPA Criteria Queries are used. The reason for the query builder
classes is that the user should not have access to the :java:extdoc:`Session<org.hibernate.Session>` object to make
sure that all query interceptors (security and more) are always applied.

Parts of the JPA Criteria API can still be used however, for example to specify conditions.
A tutorial can be found here: https://www.ibm.com/developerworks/java/library/j-typesafejpa/

.. note::
    The metamodel classes are currently not available, which means typesafe queries are not possible
    at the moment.

Implementation
--------------

ch.tocco.nice2.persist.hibernate.query.QueryBuilderBase
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`QueryBuilderBase<ch.tocco.nice2.persist.hibernate.query.QueryBuilderBase>` is the base class for all query
builders.
It contains a list of :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>` and provides several ways to add a
condition to the query:

    * Use ``QueryBuilderBase#addPredicate(Predicate)`` to add a JPA :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>` instance
    * The :java:ref:`PredicateBuilder<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder>` is a functional interface that
      can be used to create :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>` instances using lambda expressions
      that can be passed to ``QueryBuilderBase#addPredicate(PredicateBuilder)``. The :java:extdoc:`CriteriaBuilder<javax.persistence.criteria.CriteriaBuilder>`,
      :java:extdoc:`Root<javax.persistence.criteria.Root>`, :java:ref:`SubqueryFactory<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder.SubqueryFactory>`
      and the query hints are passed as parameters into the lambda expression.
    * :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` or :java:ref:`Condition<ch.tocco.nice2.persist.qb2.Condition>` instances (created by the :java:ref:`Conditions<ch.tocco.nice2.persist.qb2.Conditions>` API)
      can also be passed to ``QueryBuilderBase#addCondition()``. This API is also used by the security conditions.
      A :java:ref:`Condition<ch.tocco.nice2.persist.qb2.Condition>` is first converted into a :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>`
      instance using the :java:ref:`ConditionFactory<ch.tocco.nice2.persist.query.ConditionFactory>` and then transformed into a
      :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>` using the :java:ref:`PredicateFactory<ch.tocco.nice2.persist.hibernate.PredicateFactory>`.

It also invokes the ``QueryBuilderInterceptor#buildConditionFor()`` method of all interceptors when
the query initialization has been completed and adds the created conditions to the list of predicates.

.. note::
    This method should be called when the query builder is created; not when it is executed. For example it is expected
    that if a query that is created in privileged mode, it should remain privileged even if the privileged mode is no longer active
    when the query is executed.

The method ``QueryBuilderBase#build()`` should be called by the user when the query builder configuration is completed
and returns an object that allows to access the results. The returned object depends on the subclass and is defined by
generic parameter ``QW``.

ch.tocco.nice2.persist.hibernate.query.AbstractCriteriaBuilder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`AbstractCriteriaBuilder<ch.tocco.nice2.persist.hibernate.query.AbstractCriteriaBuilder>` is the base class
for all query builders that depend on a :java:extdoc:`CriteriaQuery<javax.persistence.criteria.CriteriaQuery>`.

It initializes a :java:extdoc:`CriteriaQuery<javax.persistence.criteria.CriteriaQuery>`, :java:extdoc:`CriteriaBuilder<javax.persistence.criteria.CriteriaBuilder>`,
:java:extdoc:`Root<javax.persistence.criteria.Root>` and :java:ref:`SubqueryFactory<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder.SubqueryFactory>`
using the ``entityClass`` (the entity that should be queried) and ``queryType`` (the result type of the query) constructor parameters.

This class also contains a map of parameters that are manually added to the query by the user and provides a helper method
to apply the parameters to the query.

Parameter handling
~~~~~~~~~~~~~~~~~~

A condition like ``field("name").is(value)`` might be mapped with a :java:extdoc:`ParameterExpression<javax.persistence.criteria.ParameterExpression>`
even though the user specified the value directly. These parameters are collected and added to the query by the :java:ref:`ParameterCollector<ch.tocco.nice2.persist.impl.qb2.ParameterCollector>`.

The parameter collector is a visitor for :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` objects. It sets an unique
name to all parameter nodes and collects their values.

The :java:ref:`ParameterCollector<ch.tocco.nice2.persist.impl.qb2.ParameterCollector>` is contained by the :java:ref:`QueryBuilderBase<ch.tocco.nice2.persist.hibernate.query.QueryBuilderBase>`
base class, because it is needed to create conditions.

.. warning::
    It is important that only one parameter collector is used per query. Otherwise the parameter names are not unique and
    the parameter values get overwritten. This means that all :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` instances
    passed to ``QueryBuilderBase#addCondition()`` must not have been already been processed by a parameter collector.

Before the query is executed the parameters collected by the :java:ref:`ParameterCollector<ch.tocco.nice2.persist.impl.qb2.ParameterCollector>`
as well as parameters that are manually passed to ``AbstractCriteriaBuilder#addParameter#addParameter()`` are applied to the
:java:extdoc:`Query<org.hibernate.query.Query>` instance (see ``AbstractCriteriaBuilder#applyParametersToQuery()``).

If the parameter value does not match the parameter type it is attempted to convert the value using ``TypeManager#convert()``.
If a :java:extdoc:`Collection<java.util.Collection>` is used as a parameter value ``Query#setParameterList()`` is used which can be
substantially faster for large parameter lists.

There are also global parameters that are applied to every query if a parameter with a certain name exists.
These are provided by the :java:ref:`ParameterProvider<ch.tocco.nice2.persist.hibernate.query.ParameterProvider>` interface.
An example would be the parameter ``currentUser`` (see :java:ref:`PrincipalNameFactory<ch.tocco.nice2.userbase.impl.ArgumentFactories.PrincipalNameFactory>`).

Subqueries
~~~~~~~~~~

The :java:ref:`AbstractCriteriaBuilder<ch.tocco.nice2.persist.hibernate.query.AbstractCriteriaBuilder>` also contains the
only implementation of the :java:ref:`SubqueryFactory<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder.SubqueryFactory>`
which can be used to create subqueries.

There are two different options:

    * ``createSubquery()`` creates a subquery that is correlated to main query (based on a given association). This can for example be used
      to create ``EXISTS`` subqueries.
    * ``createUncorrelatedSubquery()`` can be used to create any other subquery that is not correlated to the main query. The selection and
      target entity can be freely chosen.

Both methods return an instance of :java:ref:`SubqueryBuilder<ch.tocco.nice2.persist.hibernate.query.SubqueryBuilder>` which supports
similar functionality as the standard query builder.

ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`CriteriaQueryBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder>` is a base class for
'standard' query builders that expect multiple result rows and adds support for offset, limit and ordering.

Ordering
~~~~~~~~
The ordering can be defined through ``CriteriaQueryBuilder#addOrder()``. Both the JPA :java:extdoc:`Order<javax.persistence.criteria.Order>`
(can be created by the :java:extdoc:`CriteriaBuilder<javax.persistence.criteria.CriteriaBuilder>`)
and the :java:ref:`Ordering<ch.tocco.nice2.persist.query.Ordering>` class of the persist API are accepted.

Query Wrappers
~~~~~~~~~~~~~~
The :java:ref:`CriteriaQueryBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder>` defines that all
subclasses must return an implementation of :java:ref:`CriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryWrapper>`
from their ``build()`` method and provides a base implementation (``AbstractCriteriaQueryWrapper``).

It also defines the ``QT`` type parameter of its superclass to ``Object[]``. That means that the hibernate queries always
return ``Object[]`` instances. This is necessary because sometime we need to expand the user selection (see below).

The :java:ref:`CriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryWrapper>` interface defines the
following methods:

    * ``getResultList()`` returns a list of results
    * ``uniqueResult()`` returns exactly one result or null. If the query returns multiple rows, an exception will be thrown.
      Optionally a :java:extdoc:`LockModeType<javax.persistence.LockModeType>` can be passed to this method, which allows
      pessimistic locking of an entity.
    * ``distinct()`` to configure if the query should be executed with the ``DISTINCT`` keyword. The default is true.

.. note::
    Because a join in TQL is always a ``LEFT JOIN`` all standard queries need to be executed ``DISTINCT``
    to avoid duplicate results.
    However some :java:extdoc:`LockModeType<javax.persistence.LockModeType>` cause a ``SELECT FOR UPDATE`` which does not support
    distinct queries. In that case, distinct queries need to be manually disabled by calling ``distinct(false)``.

AbstractCriteriaQueryWrapper
````````````````````````````

The :java:ref:`AbstractCriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder.AbstractCriteriaQueryWrapper>`
is the base implementation of :java:ref:`CriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryWrapper>` and provides
the following functionality:

It requires a transformation :java:extdoc:`Function<java.util.function.Function>` which converts a result row (which is always
an ``Object[]``) into the desired target type.

When ``getResultList()`` is called, the following steps are taken:

    * The final ordering clause is created: If no explicit ordering is defined for the query, the default ordering defined in the entity model is used.
      In addition, the primary key is always added as the last sorting parameter (unless it already is part of the sorting clause).
      This is necessary to guarantee a consistent ordering when ``LIMIT`` or ``OFFSET`` is used (otherwise the order might be
      partially random if there are many rows with same value in the order column).
    * The final :java:extdoc:`Selection<javax.persistence.criteria.Selection>` of the query is determined: The user defined selection
      is provided by the subclass (abstract method ``getSelection()``), however it might have to be expanded:

      According to the SQL Standard all columns that are part of the ``ORDER BY`` clause must also be part of the select clause
      if it is a ``DISTINCT`` query.
      The missing columns are automatically added to the selection (``expandSelection(List<Order> order)``)
      and are removed again before the results are processed (``unwrapResults(List<Object[]> results)``).
      Due to a bug in hibernate an array selection of size 1 is not returned as array. As this breaks our code we
      add a dummy selection (the literal '1') if the the selection size is 1.

    * The :java:extdoc:`CriteriaQuery<javax.persistence.criteria.CriteriaQuery>` is then converted into a :java:extdoc:`Query<org.hibernate.query.Query>` and
      selection, conditions, ordering and parameters are applied.
    * The query is then executed and the results returned after they have been processed by the transformation function (see above).

``uniqueResult()`` works similarly, but as we expect only one result, we do not have to worry about the ordering clause.

ch.tocco.nice2.persist.hibernate.query.EntityQueryBuilder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`EntityQueryBuilder<ch.tocco.nice2.persist.hibernate.query.EntityQueryBuilder>` is an implementation
that queries for :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` instances.

It defines the :java:extdoc:`Root<javax.persistence.criteria.Root>` as the selection of the query and the mapping function
simply casts the first element of the result array into an :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>`.

ch.tocco.nice2.persist.hibernate.query.AbstractPathQueryBuilder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`AbstractPathQueryBuilder<ch.tocco.nice2.persist.hibernate.query.AbstractPathQueryBuilder>` is a base class
for query builders that use a :java:ref:`CustomSelection<ch.tocco.nice2.persist.hibernate.query.selection.CustomSelection>`.
This means that they do not return entity instances, but only certain paths.

It provides a method called ``clearSelection()`` that re-initializes the selection. However this method cannot remove joins that
were created by the previous selection and it often makes sense to just create a new query builder instance.

This class also provides the :java:ref:`CriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryWrapper>` implementation
for its subclasses: :java:ref:`CustomSelectionCriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.AbstractPathQueryBuilder.CustomSelectionCriteriaQueryWrapper>`.
``getSelection()`` returns the selection created by ``CustomSelection#toJpaSelection()`` and it overrides the methods
``internalGetResultList()`` and ``internalUniqueResult()`` and processes the query results using ``CustomSelection#mapResults()``.
This is necessary because the :java:ref:`CustomSelection<ch.tocco.nice2.persist.hibernate.query.selection.CustomSelection>`
may add additional paths (for internal processing) and some paths need to evaluated in an additional query (to-many paths for example).

ch.tocco.nice2.persist.hibernate.query.SinglePathQueryBuilder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`SinglePathQueryBuilder<ch.tocco.nice2.persist.hibernate.query.SinglePathQueryBuilder>` can be used to
query for exactly one path of an entity. The constructor takes a ``Class<T>`` parameter which defines the return type
of the query.

The ``setPath(String)`` method needs to be called to define which path should be selected.
It is verified if the selected path matches the return type, otherwise an exception will be thrown.

An exception is also thrown if ``setPath(String)`` is never called.

It returns a :java:ref:`CustomSelectionCriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.AbstractPathQueryBuilder.CustomSelectionCriteriaQueryWrapper>`
from its ``build()`` method with a mapping function that returns the first element of the result array.

ch.tocco.nice2.persist.hibernate.query.PathQueryBuilder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`PathQueryBuilder<ch.tocco.nice2.persist.hibernate.query.PathQueryBuilder>` can be used to
query for multiple paths of an entity and always returns an ``Object[]``.

The method ``addPathToSelection()`` can be called multiple times to add paths to the selection.
At least one path needs to be added otherwise an exception will be thrown.

ch.tocco.nice2.persist.hibernate.query.CriteriaCountQueryBuilder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`CriteriaCountQueryBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaCountQueryBuilder>`
executes ``COUNT`` queries and always returns a :java:extdoc:`Long<java.lang.Long>`.

It inherits directly from :java:ref:`AbstractCriteriaBuilder<ch.tocco.nice2.persist.hibernate.query.AbstractCriteriaBuilder>`
because it does not return an ``Object[]`` and also returns a different object from its ``build()`` method.

Custom Selection
----------------

The :java:ref:`CustomSelection<ch.tocco.nice2.persist.hibernate.query.selection.CustomSelection>` is used by some query builders
that select only certain paths (not entire entities).

It is not sufficient to simply add all requested paths to the JPA selection due to the following reasons:

    * Security: It must be possible to intercept field selection. The query only adds the security conditions of
      the target entity by default. But it does not check field permissions and also a path may point to a different entity
      that needs to be checked as well.
    * Paths pointing to a to-many property would return multiple rows per target entity. Even if the data would be
      merged later, it would make ``LIMIT/OFFSET`` options useless.

A custom selection contains a :java:ref:`SelectionRegistry<ch.tocco.nice2.persist.hibernate.query.selection.SelectionRegistry>`.
The selection registry keeps track of all 'requested paths' (paths that should be included in the final ``Object[]``
returned from the query builder) and all 'query paths' (paths that are included in the query).
Not all 'requested paths' will generate a 'query path' (for example to-many paths are evaluated in a separate query) and
the 'query paths' may contain additional paths that are required for internal processing, but won't be returned from the
query builder.
The selection registry maintains maps that keep track which query/requested path is at which position in the result arrays.
It also makes sure that there are no duplicated 'query paths' (for example when the same internal path is required by
multiple paths).
All the query paths can be converted into a JPA :java:extdoc:`Selection<javax.persistence.criteria.Selection>` by the
method ``toSelection()``.

The :java:ref:`CustomSelection<ch.tocco.nice2.persist.hibernate.query.selection.CustomSelection>` also contains multiple
:java:ref:`SelectionPathHandler<ch.tocco.nice2.persist.hibernate.query.selection.SelectionPathHandler>`.
A :java:ref:`SelectionPathHandler<ch.tocco.nice2.persist.hibernate.query.selection.SelectionPathHandler>` is responsible
for handling a certain type of path.

``SelectionPathHandler#processSelection()`` is called just before the JPA :java:extdoc:`Selection<javax.persistence.criteria.Selection>`
is created. The :java:ref:`SelectionRegistry<ch.tocco.nice2.persist.hibernate.query.selection.SelectionRegistry>` is passed
as an argument and can be used to add all necessary query paths to the query.

``SelectionPathHandler#processResults()`` is called after the query has been executed. Both the list of results of the query
and the target (that will be returned from the query builder) are passed as arguments. The task of the handler is to
copy the query results into the target array. The :java:ref:`SelectionRegistry<ch.tocco.nice2.persist.hibernate.query.selection.SelectionRegistry>`
contains the source and target indices of the paths.

The :java:ref:`SelectionPathHandler<ch.tocco.nice2.persist.hibernate.query.selection.SelectionPathHandler>` are also
responsible for calling the :java:ref:`QueryBuilderInterceptor<ch.tocco.nice2.persist.hibernate.query.QueryBuilderInterceptor>`
selection builder methods.

    * The :java:ref:`ToOneSelectionPathHandler<ch.tocco.nice2.persist.hibernate.query.selection.ToOneSelectionPathHandler>`
      is responsible for all 'to-one' paths. It is relatively straight-forward: the paths can be included in the query
      and after the query execution the paths can simply mapped to the target array.

    * The :java:ref:`ToManySelectionPathHandler<ch.tocco.nice2.persist.hibernate.query.selection.ToManySelectionPathHandler>`
      handles all 'to-many' paths. These paths cannot be selected directly in the query. For each base path a separate
      query is generated that retrieves the values of these paths for *all* rows. The rows are then mapped to the target array
      using the primary key of the root entity, that is selected by both queries.

    * There are special implementations for ``binary`` fields, because the ``_nice_binary`` table is not mapped by
      hibernate at the moment and cannot be queried directly. They use the :java:ref:`BinaryDataAccessor<ch.tocco.nice2.persist.hibernate.binary.BinaryDataAccessor>`
      to efficiently load :java:ref:`BinaryData<ch.tocco.nice2.persist.hibernate.binary.BinaryData>` instances, which are then merged
      into the target array.

Query Builder Interceptor
-------------------------
The :java:ref:`QueryBuilderInterceptor<ch.tocco.nice2.persist.hibernate.query.QueryBuilderInterceptor>` participates
in the query building process.

``buildConditionFor()``
^^^^^^^^^^^^^^^^^^^^^^^

This method is called for every query root and for every subquery and can add additional conditions to the query.

    - ``BusinessUnitQueryBuilderInterceptor`` makes sure that only entities belonging to the current business unit are returned
    - ``SecureQueryInterceptor`` adds additional conditions based on the security policy

The method takes an instance of :java:ref:`QueryBuilderType<ch.tocco.nice2.persist.hibernate.query.QueryBuilderInterceptor.QueryBuilderType>`
which signifies by what kind of query builder it is called. Currently ``READ`` and ``DELETE`` are supported. The
``SecureQueryInterceptor`` uses this information to apply the correct security conditions depending on the query type.

``createSelectionInterceptor()``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This method is only used when a :java:ref:`CustomSelection<ch.tocco.nice2.persist.hibernate.query.selection.CustomSelection>`
is used. It is called once for each 'base path' (a path without field) of the query.
So for example when the paths ``relUser.name``, ``relUser.lastname``, ``relAddress.address``, ``relAddress.city`` are selected,
the method is called once for ``relUser`` and ``relAddress``.

The method may return an :java:ref:`SelectionInterceptor<ch.tocco.nice2.persist.hibernate.query.QueryBuilderInterceptor.SelectionInterceptor>`,
which allows modification of the selection and inspection & replacement of the query results.

SelectionInterceptor
~~~~~~~~~~~~~~~~~~~~

``beforeQueryExecution(SelectionData)`` is called before the relevant query is executed and allows adding additional
selection paths.
One use case is to add the primary key of a 'base path' to the selection in order to be able to check access permissions.

``handleQueryResults()`` gives access to the query results and also allows overriding the query results.
The use case of the ``SecureQueryInterceptor`` is to find all primary keys of a base path using ``QueryResult#getValuesForPath()``
then check access permissions and overwrite the value with null if access is denied (using ``QueryResult#findRowsWithValueAtPath()``
and ``Row#setValueForPath()``.

Custom JDBC Functions
---------------------
Custom query functions can be implemented using the :java:ref:`JdbcFunction<ch.tocco.nice2.persist.hibernate.query.JdbcFunction>` interface.
The contributions are registered with the :java:extdoc:`SessionFactoryBuilder<org.hibernate.boot.SessionFactoryBuilder>` by the
:java:ref:`HibernateCoreBootstrapContribution<ch.tocco.nice2.persist.hibernate.bootstrap.HibernateCoreBootstrapContribution>`.

In addition to the contributed functions, the :java:ref:`GlobSqlFunction<ch.tocco.nice2.persist.hibernate.dialect.GlobSqlFunction>`
is registered as well. It implements the ``glob`` function, which is internally used when the ``Operator#LIKE`` is specified.
It uses ``LIKE`` internally but is also replacing ``*`` with ``%`` and ``?`` with ``_`` so that both placeholders are supported.

Each function must provide a :java:extdoc:`SQLFunction<org.hibernate.dialect.function.SQLFunction>` which contains the SQL template.
Typically the :java:extdoc:`SQLFunctionTemplate<org.hibernate.dialect.function.SQLFunctionTemplate>` can be used for this.
An instance of :java:ref:`SqlWriter<ch.tocco.nice2.persist.query.SqlWriter>` is provided to facilitate writing the SQL query. The
sql writer is obtained from ``Context#createSqlWriter()`` and is automatically configured based on the current :java:extdoc:`Dialect<org.hibernate.dialect.Dialect>`.

The abstract base class :java:ref:`AbstractJdbcFunction<ch.tocco.nice2.persist.hibernate.query.AbstractJdbcFunction>` provides support
to create the sql function templates:

    * Find the correct hibernate :java:extdoc:`Type<org.hibernate.type.Type>` based on the nice :java:ref:`Type<ch.tocco.nice2.types.Type>`
    * The ``writeArgument()`` method can be used to write a parameter placeholder into the sql string

.. warning::

    The arguments of the :java:ref:`Condition<ch.tocco.nice2.persist.qb2.Condition>` are passed to the criteria builder in the same order.
    If the order of arguments is different in the sql template or a parameter is used multiple times, the ``argumentOrder()`` method
    needs to be overwritten by the :java:ref:`JdbcFunction<ch.tocco.nice2.persist.hibernate.query.JdbcFunction>`. The arguments
    are then reordered and/or duplicated by the :java:ref:`FuncallArgumentProcessor<ch.tocco.nice2.persist.hibernate.pojo.CriteriaQueryCompiler.FuncallArgumentProcessor>`
    before the query is processed.

.. note::
    The :java:ref:`JdbcFunction<ch.tocco.nice2.persist.hibernate.query.JdbcFunction>` operates directly on the SQL level
    and can be used to access database specific functions.
    An example is the :java:ref:`BirthdayQueryFunction<ch.tocco.nice2.persist.backend.jdbc.impl.functions.BirthdayQueryFunction>`
    that uses the ``extract`` PostgreSQL function.

Query Functions
---------------
A :java:ref:`QueryFunction<ch.tocco.nice2.persist.spi.query.ql.QueryFunction>` can be used to implement a custom function that
can be used in the query language.
The query functions are applied by the :java:ref:`ConditionFactory<ch.tocco.nice2.persist.query.ConditionFactory>` when
the :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` tree is processed and can manipulate its nodes.

.. note::
    An example would be the :java:ref:`FulltextSearchFunction<ch.tocco.nice2.enterprisesearch.impl.queryfunction.FulltextSearchFunction>`:
    It executes the fulltext search when the query is compiled and replaces the query function node with an ``IN`` condition
    that includes the primary keys of the results of the search.

Query Compiler
--------------
The :java:ref:`CriteriaQueryCompiler<ch.tocco.nice2.persist.hibernate.pojo.CriteriaQueryCompiler>` is responsible for creating a
:java:ref:`Query<ch.tocco.nice2.persist.query.Query>` instance based on a :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>`.

The :java:ref:`QueryVisitor<ch.tocco.nice2.persist.hibernate.pojo.CriteriaQueryCompiler.QueryVisitor>` visits the node tree
and collects the entity model, condition and ordering data, which in turn will be
wrapped in a :java:ref:`HibernateQueryAdapter<ch.tocco.nice2.persist.hibernate.pojo.HibernateQueryAdapter>` that is returned
to the user.

QueryVisitor
^^^^^^^^^^^^
The query visitor handles the following funcall nodes:

    - ``Keywords.FIND``: The entity model that should be queried
    - ``Keywords.ORDER``: Each child node represents an order path and direction
    - ``Keywords.WHERE``: The condition of the query.

The condition (the WHERE part of the query) is processed by the :java:ref:`ConditionFactory<ch.tocco.nice2.persist.query.ConditionFactory>`
before it is added to the conditions list.
The condition factory applies the following visitors:

    - ``TypeSettingVisitor``: Sets the :java:ref:`Type<ch.tocco.nice2.types.Type>` of a field to the corresponding path node
    - ``QueryFunctionCompiler``: Applies all :java:ref:`QueryFunction<ch.tocco.nice2.persist.spi.query.ql.QueryFunction>` to the conditions

Predicate Factory
-----------------
The :java:ref:`PredicateFactory<ch.tocco.nice2.persist.hibernate.PredicateFactory>` converts :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` instances
representing conditions into a :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>`.
These conditions are created by the :java:ref:`QueryBuilderFactory<ch.tocco.nice2.persist.qb2.QueryBuilderFactory>`
as well as the ACL parser.

The node tree is parsed using different :java:ref:`NodeVisitor<ch.tocco.nice2.conditionals.tree.processing.NodeVisitor>`
implementations, that all extend from :java:ref:`AbstractNodeVisitor<ch.tocco.nice2.persist.hibernate.PredicateFactory.AbstractNodeVisitor>`.

AbstractNodeVisitor
^^^^^^^^^^^^^^^^^^^
This is the base class that all visitor implementations use. It defines an abstract method (``getPredicate()``) which
should return a :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>` instance for the current node.
For example the :java:ref:`LogicalNodeVisitor<ch.tocco.nice2.persist.hibernate.PredicateFactory.LogicalNodeVisitor>` converts
an :java:ref:`AndNode<ch.tocco.nice2.conditionals.tree.AndNode>`, :java:ref:`OrNode<ch.tocco.nice2.conditionals.tree.OrNode>` or
:java:ref:`NotNode<ch.tocco.nice2.conditionals.tree.NotNode>` into a :java:extdoc:`CompoundPredicate<org.hibernate.query.criteria.internal.predicate.CompoundPredicate>`.

Additionally the base class provides helper methods to handle child nodes (``handle[...]Node()``).
These helper methods create a new visitor for the given node and pass it to ``processVisitor()``, which processes the node
with the new visitor. It also calls ``Cursor#next()`` to make sure that nested calls are only handled by the newly created visitor.
Each child node is processed in isolation by its own visitor instance and its results are then aggregated by the parent visitor.

A :java:ref:`FuncallNode<ch.tocco.nice2.conditionals.tree.FuncallNode>` may be a placeholder for different types of nodes:

    - ``EXISTS`` subquery
    - ``IN`` condition
    - ``COUNT`` subquery
    - a :java:ref:`JdbcFunction<ch.tocco.nice2.persist.hibernate.query.JdbcFunction>` call

AbstractJoiningVisitor
^^^^^^^^^^^^^^^^^^^^^^
An abstract base class that handles a :java:ref:`PathNode<ch.tocco.nice2.conditionals.tree.PathNode>` and converts
the path into a :java:extdoc:`Path<javax.persistence.criteria.Path>` performing joins if necessary.

The actual work is done in :java:ref:`QueryBuilderJoinHelper<ch.tocco.nice2.persist.hibernate.QueryBuilderJoinHelper>`:

    - Iteration over all path parts (``relUser.relAddress.value`` would be three different parts)
    - If the part is an association a join to the target entity is performed
    - If it is a field, the path to that field is returned

If the path points to a primary key that is referenced in a many to one association, the foreign key field is returned
instead of performing an unnecessary join (which results in ``address.fk_user = ?`` instead of ``INNER JOIN user ON user.pk = address.fk_user WHERE user.pk = ?``
for performance reasons.

When a join is created it corresponds to an actual JOIN in the SQL. Therefore it should be tried to reuse the join instances
if the same entity is going to be joined multiple times.

RootNodeVisitor
^^^^^^^^^^^^^^^
The :java:ref:`RootNodeVisitor<ch.tocco.nice2.persist.hibernate.PredicateFactory.RootNodeVisitor>` is the entry point which handles the
root node. It simply delegates to the visitor that can handle the root node and returns the predicate of that visitor.

LogicalNodeVisitor
^^^^^^^^^^^^^^^^^^
The :java:ref:`LogicalNodeVisitor<ch.tocco.nice2.persist.hibernate.PredicateFactory.LogicalNodeVisitor>` is responsible for
handling :java:ref:`AndNode<ch.tocco.nice2.conditionals.tree.AndNode>`, :java:ref:`OrNode<ch.tocco.nice2.conditionals.tree.OrNode>`
and :java:ref:`NotNode<ch.tocco.nice2.conditionals.tree.NotNode>`.

This visitor collects all predicates of its child nodes (including other logical nodes) and nests them into an ``And``, ``Or`` or ``Not`` predicate.

ExistsNodeVisitor
^^^^^^^^^^^^^^^^^
The :java:ref:`ExistsNodeVisitor<ch.tocco.nice2.persist.hibernate.PredicateFactory.ExistsNodeVisitor>` handles
a :java:ref:`FuncallNode<ch.tocco.nice2.conditionals.tree.FuncallNode>` with the ``EXISTS`` keyword.
These nodes represent an ``EXISTS`` subquery.

The first child node is always a :java:ref:`PathNode<ch.tocco.nice2.conditionals.tree.PathNode>` that references the
relation path which is queried by the subquery. Thus the ``visitPath()`` method first creates an instance of
:java:extdoc:`Subquery<javax.persistence.criteria.Subquery>` through the :java:ref:`SubqueryFactory<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder.SubqueryFactory>`.

The path node might contain multiple relation paths which leads to nested ``EXISTS`` subqueries.
All exists predicates are collected on a stack until the path is parsed completely. The (optional)
condition is added to the top element of the stack (the one that was added last). While the predicates are removed
from the stack an exists condition is added (referencing the predicate that was removed before itself).
The last element removed from the stack is returned from the visitor.

InNodeVisitor
^^^^^^^^^^^^^
The :java:ref:`InNodeVisitor<ch.tocco.nice2.persist.hibernate.PredicateFactory.InNodeVisitor>` is used for handling
``IN`` clauses.

The values of the ``IN`` clause can either be specified as literals or parameters. The parameter names or literal values
are collected, converted to :java:extdoc:`Expression<javax.persistence.criteria.Expression>` and then passed as parameters
to an :java:extdoc:`InPredicate<org.hibernate.query.criteria.internal.predicate.InPredicate>`.

IsTrueNodeVisitor
^^^^^^^^^^^^^^^^^
The :java:ref:`IsTrueNodeVisitor<ch.tocco.nice2.persist.hibernate.PredicateFactory.IsTrueNodeVisitor>` creates a boolean
:java:extdoc:`Expression<javax.persistence.criteria.Expression>`.
Either based on a :java:extdoc:`Path<javax.persistence.criteria.Path>` that points to a boolean or a literal expression.
The latter may be used by the security framework to deny any access (``AND false``).

EquationNodeHandler
^^^^^^^^^^^^^^^^^^^
The :java:ref:`EquationNodeHandler<ch.tocco.nice2.persist.hibernate.EquationNodeHandler>` converts an
:java:ref:`EquationNode<ch.tocco.nice2.conditionals.tree.EquationNode>` into a :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>`.
An equation node consists of two nodes and an operator that defines how the two nodes can be compared.

Currently one side of the equation needs to be either a :java:extdoc:`Path<javax.persistence.criteria.Path>` or a
``COUNT`` expression.
The other side can be a literal or paramater node, another path, count expression or a jdbc function call.

If the type of the literal value does not match the type of the path or count expression, it is tried to convert
the value using the :java:ref:`TypeManager<ch.tocco.nice2.types.TypeManager>`.

The ``LIKE`` operator is handled specially as it is not translated into a SQL ``LIKE`` but mapped to our custom ``glob``
:java:extdoc:`SQLFunction<org.hibernate.dialect.function.SQLFunction>` (:java:ref:`GlobSqlFunction<ch.tocco.nice2.persist.hibernate.dialect.GlobSqlFunction>`).
Both sides of the equation are
converted to lower case to simulate ``ILIKE`` behaviour.

Localized fields
^^^^^^^^^^^^^^^^
If a localized field is part of a query it needs to be resolved for the current locale before the query is parsed.
This is achieved by the :java:ref:`EntityInterceptorVisitor<ch.tocco.nice2.persist.hibernate.pojo.EntityInterceptorVisitor>`
which is executed before the query is parsed by the predicate factory.

All path nodes are processed by the :java:ref:`FieldResolver<ch.tocco.nice2.persist.hibernate.interceptor.FieldResolver>`
and all virtual fields are replaced.

Delete query builder
====================
The :java:ref:`CriteriaDeleteBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaDeleteBuilder>` is a special query builder
implementation that can be used to delete multiple entities by query without the need to load every single entity.

The query selects the primary keys of all entities that may be deleted (the correct security conditions are added by the
``SecureQueryInterceptor``).
For each result a proxy is created, marked as deleted and the ``entityDeleting()`` event is fired. The reason for the proxy is
to avoid loading the entire entity unless it is absolutely necessary (for example when the entity data is accessed by a listener).

Note that ``Entity#markDeleted()`` is used. This is an internal method that can be invoked without initializing the proxy
(as opposed to ``delete()``) and causes ``getState()`` to correctly return ``PHANTOM``.

After the invocation of the listeners the proxy instances are scheduled for deletion with the :java:ref:`EntityTransactionContext<ch.tocco.nice2.persist.hibernate.cascade.EntityTransactionContext>`.
Note that the ``addDeletedEntityBatch()`` method is used that deletes the entire batch with one delete statement (as opposed to
the normal behaviour which fires a delete statement for every deleted entity).