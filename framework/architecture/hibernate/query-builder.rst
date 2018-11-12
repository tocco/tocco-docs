Query Builder
=============

Queries can be executed through the :java:ref:`CriteriaQueryBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder>`,
which can be obtained from ``PersistService#createQueryBuilder(java.lang.Class<T>)``.
The argument passed to that method defines which entity should be queried.
The query builder can also be obtained from a second method (``PersistService#createQueryBuilder(java.lang.Class<?>, java.lang.Class<T>)``)
that needs to be used if a custom selection is specified (e.g. only some fields should be returned and not the entire entity) and the return type is not the same as the entity that is being queried.

Internally, JPA Criteria Queries are used. The reason for the :java:ref:`CriteriaQueryBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder>`
class is that the user should not have access to the :java:extdoc:`Session<org.hibernate.Session>` object to make
sure that all query interceptors (security and more) are always applied.

Parts of the JPA Criteria API can still be used however, for example to define the selection or to specify conditions.
A tutorial can be found here: https://www.ibm.com/developerworks/java/library/j-typesafejpa/

.. note::
    The metamodel classes are currently not available, which means typesafe queries are not possible
    at the moment.

Selection
---------
If nothing is specified explicitly an instance of the specified entity class (or a list thereof) is returned from the query.
But it is also possible to specify a custom selection (single fields, functions like ``avg`` and so on).
The selection can be built with the JPA Criteria API using the :java:extdoc:`CriteriaBuilder<javax.persistence.criteria.CriteriaBuilder>` and the
:java:extdoc:`Root<javax.persistence.criteria.Root>` which can be obtained from the query builder.

Conditions
----------
There are several ways to add a condition to the query:

Predicate
^^^^^^^^^
It is possible to add a JPA :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>` by calling
``CriteriaQueryBuilder#addPredicate()``.
The Predicate can be constructed using the :java:extdoc:`CriteriaBuilder<javax.persistence.criteria.CriteriaBuilder>`,
:java:extdoc:`Root<javax.persistence.criteria.Root>` and :java:ref:`SubqueryFactory<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder.SubqueryFactory>`
that can be obtained from the CriteriaQueryBuilder.

.. note::
    The ``SubqueryFactory`` can be used to create :java:extdoc:`Subquery<javax.persistence.criteria.Subquery>` instances.
    It was introduced to hide the query internals from the user and to make sure that the interceptors cannot be bypassed.
    It is primarily used to create ``EXISTS`` subqueries.
    The factory automatically calls the :java:ref:`QueryBuilderInterceptor<ch.tocco.nice2.persist.hibernate.query.QueryBuilderInterceptor>`
    for the subquery type and correlates the subquery with the related association.

PredicateBuilder
^^^^^^^^^^^^^^^^
The :java:ref:`PredicateBuilder<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder>` is a functional interface that
can be used to create :java:extdoc:`Predicate<javax.persistence.criteria.Predicate>` instances using lambda expressions
that can be passed to ``CriteriaQueryBuilder#addPredicate()``. The :java:extdoc:`CriteriaBuilder<javax.persistence.criteria.CriteriaBuilder>`,
:java:extdoc:`Root<javax.persistence.criteria.Root>` and :java:ref:`SubqueryFactory<ch.tocco.nice2.persist.hibernate.query.PredicateBuilder.SubqueryFactory>`
are passed as parameters into the lambda expression.

``Node`` API
^^^^^^^^^^^^
:java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` instances created by the :java:ref:`Conditions<ch.tocco.nice2.persist.qb2.Conditions>` API
can also be passed to ``CriteriaQueryBuilder#addCondition()``. This API is also used by the security conditions.

A condition like ``field("name").is(value)`` might be mapped with a :java:extdoc:`ParameterExpression<javax.persistence.criteria.ParameterExpression>`
even though the user specified the value directly. These parameters are collected and added to the query by the :java:ref:`ParameterCollector<ch.tocco.nice2.persist.impl.qb2.ParameterCollector>`.

The parameter collector is a visitor for :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` objects. It sets an unique
name to all parameter nodes and collects their values. The result can then be used to set the parameter values to the query.

.. warning::
    It is important that only one parameter collector is used per query. Otherwise the parameter names are not unique and
    the parameter values get overwritten. This means that all :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` instances
    passed to ``CriteriaQueryBuilder#addCondition()`` must not have been already be processed by a parameter collector.

Ordering
--------
The ordering can be defined through ``CriteriaQueryBuilder#addOrder()``. Both the JPA :java:extdoc:`Order<javax.persistence.criteria.Order>`
(can be created by the :java:extdoc:`CriteriaBuilder<javax.persistence.criteria.CriteriaBuilder>`)
and the :java:ref:`Ordering<ch.tocco.nice2.persist.query.Ordering>` class of the persist API are accepted.

Query Wrappers
--------------
When the query builder configuration (selection, conditions etc) is finished, an instance of :java:ref:`CriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryWrapper>`
can be created by calling ``build()``.

The query is not executed yet.

.. note::
    The query interceptors are applied during the configuration phase and not when the query is actually executed!

The query wrapper wraps an instance of :java:extdoc:`CriteriaQuery<javax.persistence.criteria.CriteriaQuery>`.
The wrapper configures the query instance (selection, predicates, ordering etc) and reuses it for different
purposes (``getResultList()`` or ``count()``).

If the parameter values do not match the referenced type it is tried to convert them using the
:java:ref:`TypeManager<ch.tocco.nice2.types.TypeManager>`.

Ordering
^^^^^^^^
If no explicit ordering is defined for the query, the default ordering defined in the entity model is used.
According to the SQL Standard all columns that are part of the ``ORDER BY`` clause must also be part of the select clause
if it is a ``DISTINCT`` query.

If the user specifies a custom selection, it is his responsibility to build a valid query.
But for 'normal' queries that select entire entities, this should be handled automatically.
This is the case if the ordering contains a column of a different entity.
The additional columns are added to the selection are discarded again when the results are processed
(see :java:ref:`EntityCriteriaQueryWrapper<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder.EntityCriteriaQueryWrapper>`).


Query Builder Interceptor
-------------------------
The :java:ref:`QueryBuilderInterceptor<ch.tocco.nice2.persist.hibernate.query.QueryBuilderInterceptor>` participates
in the query building process.
It is called for every query root and for every subquery and can add additional conditions to the query.

    - ``BusinessUnitQueryBuilderInterceptor`` makes sure that only entities belonging to the current business unit are returned
    - ``SecureQueryInterceptor`` adds additional conditions based on the security policy

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
The query functions are applied when the :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>` tree is processed
and can manipulate the tree.

.. note::
    An example would be the :java:ref:`FulltextSearchFunction<ch.tocco.nice2.enterprisesearch.impl.queryfunction.FulltextSearchFunction>`:
    It executes the fulltext search when the query is compiled and replaces the query function node with an ``IN`` condition
    that includes the primary keys of the results of the search.

Query Compiler
--------------
The :java:ref:`CriteriaQueryCompiler<ch.tocco.nice2.persist.hibernate.pojo.CriteriaQueryCompiler>` is responsible for creating a
:java:ref:`Query<ch.tocco.nice2.persist.query.Query>` instance based on a :java:ref:`Node<ch.tocco.nice2.conditionals.tree.Node>`.

The :java:ref:`QueryVisitor<ch.tocco.nice2.persist.hibernate.pojo.CriteriaQueryCompiler.QueryVisitor>` visits the node tree
and creates a :java:ref:`CriteriaQueryBuilder<ch.tocco.nice2.persist.hibernate.query.CriteriaQueryBuilder>`, which in turn will be
wrapped in a :java:ref:`HibernateQueryAdapter<ch.tocco.nice2.persist.hibernate.pojo.HibernateQueryAdapter>` that is returned
to the user.

QueryVisitor
^^^^^^^^^^^^
The query visitor handles the following funcall nodes:

    - ``Keywords.FIND``: The entity model that should be queried
    - ``Keywords.ORDER``: Each child node represents an order path and direction
    - ``Keywords.WHERE``: The condition of the query.

The condition (the WHERE part of the query) is passed to ``CriteriaQueryBuilder#addCondition()`` where it is processed by the :java:ref:`PredicateFactory<ch.tocco.nice2.persist.hibernate.PredicateFactory>`.
The node gets processed by the following visitors before it is passed to the query builder:

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
If there are nested ``EXISTS`` subqueries, the inner one is set as the condition of the outer one:

.. code:: java

    if (subquery != null) {
        subquery.where(criteriaBuilder.exists(newSubquery));
    }

The predicate that is returned from this visitor is always the outermost ``EXISTS`` predicate.

After the subqueries have been created the (optional) condition is parsed and added to the innermost subquery
as additional condition.

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