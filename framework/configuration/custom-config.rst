Custom Configurations
=====================

Environment Variable
--------------------

The ``ch.tocco.nice2.runenv`` VM parameter accepts ``development``, ``test`` and ``production`` as value.
In the code the value can be requested over ``Nice2.getRunEnv()``.
In a production system an error can only be logged while during development an exception should be thrown.

Application Properties
----------------------

You can override the customer specific application properties in the file ``customer/NAME/etc/application.local.properties``.
This file is not tracked in the VCS and you override the properties of the ``application.properties`` file.

Here are some examples of what you can configure:

* Set ``nice2.entities.minimal=true`` that only required entities are loaded. This rule is used in combination with:

.. code-block:: bash

    model.entity.object.pattern=(?:Address|Principal|BatchJob)(?!.*remove).*\.xml$
    model.entity.relation.pattern=Address_to_Principal(?!.*remove).*\.xml$
    model.form.pattern=(?:Address|Principal)(?!.*remove).*\.xml$

* If the CMS is not used it can be disabled by ``nice2.cms.use=false``.

* Sometimes it might be helpful to disable the foreign key validation during the development. With ``nice2.validate.model=false`` it can be disabled.

* Per default HTTP-Connections are only allowed by localhost. With ``hiveapp.http.host=0.0.0.0`` the configuration can be overwritten.

* With ``nice2.usermanager.generatelogin.disabled=true`` the ``GenerateLoginEntityListener`` can be disabled.

VM-Parameters
-------------

* ``-Dch.tocco.nice2.disableRoleSync=true`` the role check is not executed
* ``-Dch.tocco.nice2.disableLanguageSync=true`` the language check is disabled
* ``-Dch.tocco.nice2.enterprisesearch.disableStartup=true`` the search index is disabled
* ``-Dch.tocco.nice2.reporting.disableStartupValidation=true`` reports are not validated and compiled
* ``-Dch.tocco.nice2.optional.ldapserver.disableStartup=true`` the LDAP server is not started
* ``-Dch.tocco.nice2.disableStartupJsFileGeneration=true`` the Javascript files are not directly generated
* ``-Dch.tocco.nice2.disableSchemaModelStartupCheck=true`` the DB-schema is not automatically checked
* ``-Dch.tocco.nice2.cms.template.synchronize.enable=true`` CMS templates are synchronized at the application startup. If ``runenv=development`` it disabled per default
* ``-Dch.tocco.nice2.reporting.synchronize.enable=true`` reports are synchronized at the application startup. If ``runenv=development`` it disabled per default
* ``-Dch.tocco.nice2.enableUpgradeMode=true`` the application is started in the upgrade mode. Only used by the continuous delivery pipeline

If a new database is created the following parameters are required:

.. code-block:: bash

    -Dch.tocco.nice2.enterprisesearch.disableStartup=true
    -Dch.tocco.nice2.enableUpgradeMode=true
    -Dch.tocco.nice2.reporting.disableStartupValidation=true