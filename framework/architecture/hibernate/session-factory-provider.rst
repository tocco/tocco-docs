Hibernate Setup
===============

Introduction
------------
Hibernate is an implementation of the JPA specification, however we use the Hibernate API directly in many cases
(instead of using the JPA API in the :java:extdoc:`javax.persistence` package), mostly because we need many custom Hibernate
features to be able to provide exactly the same behaviour as the old Persistence API did.

Build the SessionFactory
------------------------
The first step is to initialize a :java:extdoc:`SessionFactory<org.hibernate.SessionFactory>`.
This is done by the :java:ref:`SessionFactoryProvider<ch.tocco.nice2.persist.hibernate.bootstrap.SessionFactoryProvider>`,
which is a hivemind :java:ref:`ServiceImplementationFactory<org.apache.hivemind.ServiceImplementationFactory>`. This is a sort of a factory service that can be used to
inject the generated object (in this case the :java:extdoc:`SessionFactory<org.hibernate.SessionFactory>`) into other services.

The Hibernate bootstrapping process is documented `here <http://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#bootstrap-native>`_.

This service is registered with private visibility and can only be used in the ``persist/core`` module, because if other modules
would use the :java:extdoc:`SessionFactory<org.hibernate.SessionFactory>` directly, they could bypass the security layer.

Participate in the bootstrap process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible for other hivemind modules to apply custom configuration options during the building of the
session factory by contributing a :java:ref:`HibernateBootstrapContribution<ch.tocco.nice2.persist.hibernate.HibernateBootstrapContribution>`.
This contribution provides several methods to participate in various steps of the bootstrapping process. It's also possible
to provide a priority to control the execution order of the different contributions.
This can for example be used for registering custom user types.

The main configuration is done by :java:ref:`HibernateCoreBootstrapContribution<ch.tocco.nice2.persist.hibernate.bootstrap.HibernateCoreBootstrapContribution>`.

ContributionClassLoaderService
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :java:ref:`ContributionClassLoaderService<ch.tocco.nice2.persist.hibernate.ContributionClassLoaderService>` is a custom
:java:extdoc:`ClassLoaderService<org.hibernate.boot.registry.classloading.spi.ClassLoaderService>` which makes it easy
to contribute services at runtime and to avoid having to use the :java:extdoc:`ServiceLoader<java.util.ServiceLoader>`
API used by the default implementation.

Bootstrap steps
---------------

Generate entity classes
^^^^^^^^^^^^^^^^^^^^^^^

Entity classes are generated based on the entity models and then registered
with the provided :java:extdoc:`MetadataSources<org.hibernate.boot.MetadataSources>`.

See :doc:`entity-class-generation`.

.. todo::
   Describe other steps