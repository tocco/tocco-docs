Hibernate Setup
===============

Introduction
------------
Hibernate is an implementation of the JPA specification, however we use the Hibernate API directly in many cases
(instead of using the JPA API in the `javax.persistence <https://javaee.github.io/javaee-spec/javadocs/javax/persistence/package-summary.html>`__ package), mostly because we need many custom Hibernate
features to be able to provide exactly the same behaviour as the old Persistence API did.

Build the SessionFactory
------------------------
The first step is to initialize a :java-hibernate:`SessionFactory <org/hibernate/SessionFactory>`.
This is done by the :nice:`SessionFactoryProvider <ch/tocco/nice2/persist/hibernate/bootstrap/SessionFactoryProvider>`,
which is a hivemind :abbr:`ServiceImplementationFactory (org.apache.hivemind.ServiceImplementationFactory)`. This is a sort of a factory service that can be used to
inject the generated object (in this case the :java-hibernate:`SessionFactory <org/hibernate/SessionFactory>`) into other services.

The Hibernate bootstrapping process is documented `here <https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#bootstrap-native>`_.

This service is registered with private visibility and can only be used in the ``persist/core`` module, because if other modules
would use the :java-hibernate:`SessionFactory <org/hibernate/SessionFactory>` directly, they could bypass the security layer.

.. _bootstrap:

Participate in the bootstrap process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible for other hivemind modules to apply custom configuration options during the building of the
session factory by contributing a :nice:`HibernateBootstrapContribution <ch/tocco/nice2/persist/hibernate/HibernateBootstrapContribution>`.
This contribution provides several methods to participate in various steps of the bootstrapping process. It's also possible
to provide a priority to control the execution order of the different contributions.
This can for example be used for registering custom user types.

The main configuration is done by :nice:`HibernateCoreBootstrapContribution <ch/tocco/nice2/persist/hibernate/bootstrap/HibernateCoreBootstrapContribution>`.

.. _classLoaderService:

ContributionClassLoaderService
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :nice:`ContributionClassLoaderService <ch/tocco/nice2/persist/hibernate/ContributionClassLoaderService>` is a custom
:java-hibernate:`ClassLoaderService <org/hibernate/boot/registry/classloading/spi/ClassLoaderService>` which makes it easy
to contribute services at runtime and to avoid having to use the :java:`ServiceLoader <java/util/ServiceLoader>`
API used by the default implementation.

Bootstrap steps
---------------

Register custom user extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Several user extensions are registered with the :nice:`ContributionClassLoaderService <ch/tocco/nice2/persist/hibernate/ContributionClassLoaderService>`:

    - For each custom user type a :java-hibernate:`TypeContributor <org/hibernate/boot/model/TypeContributor>` is contributed.
      There are some default types (for example ``binary`` or ``datetime``) that are always registered, but other modules can
      contribute user types as well (see :ref:`user-types`).
    - :nice:`FieldGenerator <ch/tocco/nice2/persist/hibernate/pojo/FieldGenerator>` contributions (fields that are set
      automatically by the framework, like the create/update timestamps and users) (see :ref:`generated-values`).

Generate entity classes
^^^^^^^^^^^^^^^^^^^^^^^

Entity classes are generated based on the entity models and then registered
with the provided :java-hibernate:`MetadataSources <org/hibernate/boot/MetadataSources>`.

See :doc:`entity-class-generation`.

Apply Hibernate properties
^^^^^^^^^^^^^^^^^^^^^^^^^^

The next step is to apply the Hibernate configuration settings.
The interface :nice:`HibernatePropertiesProvider <ch/tocco/nice2/persist/hibernate/HibernatePropertiesProvider>`
defines some common properties in a default method.

The only implementation (:nice:`HibernatePropertiesProviderImpl <ch/tocco/nice2/persist/hibernate/bootstrap/HibernatePropertiesProviderImpl>`)
adds the connection options to the default properties. These are read from the different ``hikaricp.properties`` files
(base, customer and local).
The properties need to be transformed to a different format as Hibernate uses different options than HikariCP.

The :nice:`ToccoDialectResolver <ch/tocco/nice2/persist/hibernate/dialect/ToccoDialectResolver>` is a custom
:java-hibernate:`DialectResolver <org/hibernate/engine/jdbc/dialect/spi/DialectResolver>`, which makes sure that our custom dialects are used
by hibernate. It is configured using the ``hibernate.dialect_resolvers`` property.

Injecting service factories
^^^^^^^^^^^^^^^^^^^^^^^^^^^

We use a custom implementation of :java-hibernate:`PersisterFactory <org/hibernate/persister/spi/PersisterFactory>`.
This allows (manually) injecting hivemind services or contributions into a custom persister or dialect.
Without using a custom factory, Hibernate just calls the default constructor.

Hibernate interceptor
^^^^^^^^^^^^^^^^^^^^^

A custom Hibernate :java-hibernate:`Interceptor <org/hibernate/Interceptor>` is registered as well.
In order to be able to split up the functionality of the interceptor into different classes
(perhaps from different modules) the :nice:`DelegatingHibernateInterceptor <ch/tocco/nice2/persist/hibernate/listener/DelegatingHibernateInterceptor>`
is used (as it is not possible to register multiple interceptors). This class then delegates the events to the
actual interceptor implementations.

Currently only one interceptor is used:

    - :nice:`ValidationInterceptor <ch/tocco/nice2/persist/hibernate/validation/ValidationInterceptor>` which runs the
      entity validation before the changes are flushed to the database.

JDBC function registration
^^^^^^^^^^^^^^^^^^^^^^^^^^

All :nice:`JdbcFunction <ch/tocco/nice2/persist/hibernate/query/JdbcFunction>` are registered with the
:java-hibernate:`SessionFactoryBuilder <org/hibernate/boot/SessionFactoryBuilder>`.

Event listener registration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Multiple Hibernate listeners (see :java-hibernate:`EventType <org/hibernate/event/spi/EventType>`) are registered:

    - :nice:`ExtendedInitializeCollectionEventListener <ch/tocco/nice2/persist/hibernate/interceptor/ExtendedInitializeCollectionEventListener>`
      initializes collections using a custom query which includes security and business unit predicates. See :doc:`collections`.
    - :abbr:`CustomDeleteEventListener (ch.tocco.nice2.persist.hibernate.cascade.CustomDeleteEventListener)` makes sure
      that deleted entities are automatically removed from many to many associations (see :ref:`delete_event_listener`).
    - :nice:`CustomFlushEntityEventListener <ch/tocco/nice2/persist/hibernate/listener/CustomFlushEntityEventListener>` handles
      custom after commit events (see :ref:`flush_event`)
    - :nice:`AfterCommitListener <ch/tocco/nice2/persist/hibernate/listener/AfterCommitListener>` and
      :nice:`CustomFlushEntityEventListener <ch/tocco/nice2/persist/hibernate/listener/CustomFlushEntityEventListener>`
      are responsible for firing after commit events (see :ref:`Listeners`).

Startup time improvements
^^^^^^^^^^^^^^^^^^^^^^^^^

Hibernate completely initializes every entity during the construction of the session factory.
Among many other things this includes:

    - A :java-hibernate:`ProxyFactory <org/hibernate/proxy/ProxyFactory>` for every entity (required to instantiate
      lazily loaded entity proxies). These are currently based on javassist and take some time to initialize,
      especially for hundreds of entities.
    - Several :java-hibernate:`UniqueEntityLoader <org/hibernate/loader/entity/UniqueEntityLoader>` per entity
      (one per :java-hibernate:`LockMode <org/hibernate/LockMode>`). Apart from the fact that we don't need all lock modes,
      they are also expensive to initialize because they contain the SQL string required to load the entity.

This makes sense for a production environment, but during development a quicker startup time is more important because
usually only a fraction of all entities is used. It therefore makes more sense to initialize these objects on the fly when
they are needed for the first time.

To support this we use the :nice:`CustomEntityPersister <ch/tocco/nice2/persist/hibernate/CustomEntityPersister>` that
returns a custom lazy implementation of :java-hibernate:`UniqueEntityLoader <org/hibernate/loader/entity/UniqueEntityLoader>`
which is not initialized until it is needed.

Similarly, the :nice:`CustomEntityTuplizer <ch/tocco/nice2/persist/hibernate/CustomEntityTuplizer>` does not initialize
the :java-hibernate:`ProxyFactory <org/hibernate/proxy/ProxyFactory>` until it is needed.
