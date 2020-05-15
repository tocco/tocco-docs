Custom entity tuplizer
======================

We use a custom :java-hibernate:`EntityTuplizer <org/hibernate/tuple/entity/EntityTuplizer>`
(:abbr:`CustomEntityTuplizer (ch.tocco.nice2.persist.hibernate.CustomEntityTuplizer)`) to support the following features:

Lazy initialization of proxy factories
--------------------------------------

Hibernate creates a :java-hibernate:`ProxyFactory <org/hibernate/proxy/ProxyFactory>` for each entity class.
To improve the startup time of the application we create them lazily when they are needed for the first time.
In addition this saves some memory as, most likely, proxies are not created for every entity class we use.

This is achieved by overriding ``buildProxyFactory()`` and returning a proxy instance that is initialized when it is used for the
first time.

Custom proxy factory
--------------------

We use our own custom :java-hibernate:`JavassistProxyFactory <org/hibernate/proxy/pojo/javassist/JavassistProxyFactory>`
(:abbr:`NiceJavassistProxyFactory (ch.tocco.nice2.persist.hibernate.proxy.NiceJavassistProxyFactory)`) that uses
the :abbr:`NiceJavassistLazyInitializer (ch.tocco.nice2.persist.hibernate.proxy.NiceJavassistLazyInitializer)` as
:java-hibernate:`LazyInitializer <org/hibernate/proxy/LazyInitializer>`.

The lazy initializer contains the entity class and the identifier and by default initializes (i.e loads the data from the
database) when any method is called on the proxy.
However many methods of the :abbr:`Entity (ch.tocco.nice2.persist.entity.Entity)` interface do not require the information
from the database and would trigger an unnecessary proxy initialization.

Our custom lazy initializer prevents this and evaluates certain methods without initializing the proxy:

    * ``requireKey()`` / ``getKey()``: the identifier is contained by the initializer and can be returned as :abbr:`PrimaryKey (ch.tocco.nice2.persist.entity.PrimaryKey)`
    * ``hasKey()``: always returns true because proxies always belong to persistent entities
    * ``getContext()`` / ``getManager()`` / ``getModel()``: the :abbr:`DataModel (ch.tocco.nice2.persist.model.DataModel)`
      and :abbr:`Context (ch.tocco.nice2.persist.Context)` are injected into our custom initializer and can be returned directly or
      used in combination with the persistent class to evaluate these methods

Some additional methods are evaluated directly by the initializer only when the proxy is not initialized yet:

    * ``getState()`` is always ``CLEAN`` when the data was not loaded yet
    * ``isFieldChanged()`` / ``isFieldTouched()`` / ``getChangedFields()`` / ``getTouchedFields()`` / ``getTouchedRelations()``:
      No fields are changed before the proxy is loaded

The custom initializer also plays a role when using the ``EntityManager#delete(Condition)`` method, that deletes a number of entities
without initializing them (if possible). For that reason ``markDeleted()`` is implemented on the initializer as well
so that ``getState()`` can correctly return ``PHANTOM``.



