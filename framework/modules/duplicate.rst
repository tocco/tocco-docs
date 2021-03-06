Duplicate
=========

Config
------

The way duplicate entities are identified can be configured in the entity ``Duplicate_config``. ``Duplicate_field``
entities are grouped into ``Duplicate_field_group`` entities. Fields in a group are combined as AND conditions, while
groups themselves are combined as OR conditions. For each ``Duplicate_field`` a field to check and the
``Duplicate_comparison_type`` have to be set. Currently there exist two options:

* ``exact``
    * check fields for equality of their values
* ``fuzzy``
    * check fields for the similarity of their values by comparing their trigram scores
    * by default a similarity of 0.6 is required (see documentation of ``pg_trgm`` for what that exactly means)
    * required similarity can be set in ``nice2.duplicate.min.trigramSimilarity`` in ``application.properties``

Configs can be activated and deactivated through the actions on their list and detail forms.
After activating a config, a search for duplicates in the system is started. The results can be found
as ``Duplicate`` entities. These are kept up to date as long as the config is active. When a config is deactivated,
all duplicates generated by that config are deleted.

For each entity model, a filter can be defined in ``hivemodule.xml`` by setting a relation and unique ids to look for.
Entities that contain any of the ids in the defined relation are excluded from the duplicate search. See example at the
bottom of this page.

At this moment, ``Duplicate_config`` can only be created for ``User`` and ``Address``.

Enabling duplicate search for new entity models
-----------------------------------------------

.. note::

        Check the examples below if one of the steps is unclear.

#. Add dependency on ``nice2.duplicate`` to ``hivemodule.xml`` of the new entity model.
#. Define relation to filter new entity model by in ``hivemodule.xml``.
#. Add initial values for ``Duplicate_entity_model`` for the new entity model.
#. Create relations from ``Duplicate`` to new entity model, like ``Duplicate_relAddress``.
#. Add the above relation to the detail form of the new entity model.
#. Deny all write rights on this new relation.
#. Add rights for manager role to see duplicates.
#. Set :nice:`DuplicateListener <ch/tocco/nice2/duplicate/impl/search/DuplicateListener>` to run for new entity model in ``hivemodule.xml``.
#. Create ``Duplicate_config``, ``Duplicate_field_group`` and ``Duplicate_field`` entities.
#. Activate the new ``Duplicate_config`` once you're satisfied with it.

.. code-block:: yaml
        :caption: Initial values example for ``User``

        Duplicate_entity_model:
          localized:
            label: entities.User
          fields:
            unique_id: user
            sorting: 10
            active: true
          config:
            identifier: unique_id
            priority: 30

.. code-block:: xml
        :caption: Field example for ``Duplicate_relUser``

        <field data="relDuplicate" scopes="read,update" ignore-copy="true"/>

.. code-block:: xml
        :caption: Relation example for ``User``

        <?xml version="1.0" encoding="UTF-8"?>
        <relation xmlns="http://nice2.tocco.ch/schema/relation.xsd">
          <source entity-model="Duplicate">
            <delete cascade="no"/>
          </source>
          <target entity-model="User">
            <delete cascade="no"/>
          </target>
          <cardinality>n:n</cardinality>
        </relation>

.. code-block:: none
        :caption: ACL example for ``User``

        entityPath(Duplicate, relUser):
            deny access(write);

        entityPath(User, relDuplicate):
            deny access(write);

        entity(Duplicate):
            grant access to usermanager;

.. code-block:: xml
        :caption: Hivemodule example for ``User``

        <dependency module-id="nice2.duplicate"/>

        <contribution configuration-id="nice2.duplicate.DuplicateFilters">
          <filter model="User" lookup-relation="relUser_status" excluded-values="archive"/>
        </contribution>

        <contribution configuration-id="nice2.persist.core.EntityListeners">
          <listener listener="service:nice2.duplicate.DuplicateListener" filter="User"/>
        </contribution>
