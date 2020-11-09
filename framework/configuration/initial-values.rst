.. _InitialValues:

Initial Values
==============

Entities can be defined as initial values in ``YAML`` files. These will be created when initializing a database and
will be updated in the database when they differ from each other, depending on how they are configured.

.. note::

    To be able to create entities through initial values they need to be uniquely identifiable through some field. This
    must not be the primary key, as that is handled by the database directly and should not be set to a fixed value.

File location
-------------

Initial values are placed in the folder ``<path of module>/module/db/initialvalues``. No other folders or
subfolders are checked, so make sure to place them in the correct folder. The filename must end in ``.yaml``, the rest
of the filename does not matter. Our suggestion is to create one file for each entity model that you are trying to
create and naming the file after that entity model.

Format
------

All possible configurations and values can be seen in the following example. Everything is optional and will be replaced
by a default value. Check the corresponding documentations under :ref:`Configuration` for its function and default value.

.. code:: yaml

    Entity_model:
        config:
            identifier: some_identifier # string, a field name
            update-mode: force_update | create_only | keep_changes
            business-unit-mode: create_for_all | use_field_value
            priority: 10
            extension: false
        fields:
            some_identifier: identifier
            sorting: 10
            active: true
        localized:
            label: entities.Entity_model.identifer # string, a text resource key
        relations:
            fk_other_entity: nice_other_entity WHERE unique_id = 'identifier' # string, table name and condition of a SQL query (should return single result)
        single-relations:
            relOther_entity: nice_other_entity WHERE unique_id = 'identifier' # string, relation name and condition of a SQL query (should return single result)
        multi-relations:
            fk_other_entity: nice_other_entity WHERE unique_id IN ('identifier', 'some-other-id') # string, relation name and condition of a SQL query (can return many result)

.. _Configuration:

Configuration
-------------

:ref:`fields`, :ref:`localized`, :ref:`relations`, :ref:`single-relations` and :ref:`multi-relations` are used to define
the values to be written into the database. Any field or relation that does not exist in the database schema will be
ignored. If your record has no localized fields or relations, you may omit those parts completely.

:ref:`config` is used to define how the record is created or updated. A default value exists for all configurations.

.. _config:

config
^^^^^^

.. _identifier:

identifier
""""""""""

Define the field that can be used to identify a single record. This is used to find existing records to update. The
field that is defined here always needs to have a value defined under :ref:`fields`. Default is ``unique_id``.

update-mode
"""""""""""

Define the mode that should be used to update existing records. Default is ``keep_changes``.

* force_update
    Always replace values in fields with those defined in the initial values, even if they were changed on the database.
* create_only
    Never update any fields in a existing record.
* keep_changes
    Checks that the ``_nice_version`` field of the existing record equals ``1``. If it is, update all fields to the
    value defined in the initial values. Otherwise, do nothing.

Entities that do not use the nice version field, will logically also not be checked for any version changes. Your
initial values will therefore always be updated, unless it is configured as a ``create_only`` value.

.. note::

    Currently all initial values are defined as ``keep_changes``, as we do not want to ever overwrite customer specific
    changes.

business-unit-mode
""""""""""""""""""

Define the mode that should be used when handling entities with business unit dependency. Default is ``use_field_value``.

* create_for_all
    Create a record for each business unit in the database. Existing records are then identified by the combination of
    the identifier field value and the relation to the business unit.
* use_field_value
    Do not set the business unit relation automatically, but use whatever was defined in :ref:`relations`,
    :ref:`fields`, :ref:`single-relations` or :ref:`multi-relations`.

priority
""""""""

This can be used when you need to ensure that certain initial values will be created in a given order. Initial values
will be sorted descending by the priority and then run in sequence. Any integer is a valid value. Default is ``0``.

In any given priority the initial values are sorted by their modules with full consideration for dependencies between
modules. So you should be able to ignore the priority completely if you only depend on an initial values from a module
you have a dependency on.

extension
"""""""""

Set this to ``true`` if you need to extend an existing initial value from another module. A usual use case will be to
add values for fields or relations that have been added in later modules. To extend another value, the :ref:`identifier`
configuration and the identifier value itself of the source and extension value need to match.

You are able to overwrite values form the source, but use this with caution. There is no guarantee for a deterministic
result if multiple extensions overwrite the same values. It will *probably* be the last loaded module that wins out, but
this is not actually verified anywhere.

You are not able to extend or overwrite anything from :ref:`config`.

.. _fields:

fields
^^^^^^

Define any static values for fields here. The key always corresponds to a field on the database. The fields
``_nice_version``, ``_nice_create_user``, ``_nice_create_timestamp``, ``_nice_update_user`` and
``_nice_update_timestamp`` can not be used here, as they are inserted dynamically depending on whether you are creating
or updating a record.

.. warning::

    It is possible to set values for localized fields and relations here. But this will seldomly be what you want.

    Use :ref:`localized` to define values in properly localized text resources, instead of having to set each field for
    each locale by yourself.

    Use :ref:`relations`, :ref:`single-relations` or :ref:`multi-relations` to dynamically find keys for related
    entities, instead of having to set fixed keys here.

.. _localized:

localized
^^^^^^^^^

Define the text resources where localized values should be taken from. The key should be the name of the localized field
**without** the locale part. For example, if your ``label`` field is localized, you'd use ``label`` as the key here,
**not** ``label_de``, ``label_en``, etc. The value will be interpreted as a text resource key and read in each locale
that is installed on the system. See :doc:`textresources`.

.. _relations:

relations
^^^^^^^^^

Define queries that should be used to fill n:1 or n:0..1 relations. The key always corresponds to a field on the
database. The value will be used as part of a SQL query to determine the key that should be written to the field. Your
value should contain the table name of the target table and a ``WHERE`` condition that uniquely identifies a single
record.

.. code-block:: sql
   :caption: Example

    -- value as defined in initial value
    fk_target_table: nice_target_table WHERE unique_id = 'identifier'

    -- query that will be executed on the database
    SELECT pk FROM nice_target_table WHERE unique_id = 'identifier'

.. _single-relations:

single-relations
^^^^^^^^^^^^^^^^

Works the exact same way as :ref:`relations`, but you can use the relation name from Nice as the field name.

.. code-block:: sql
   :caption: Example for a relation Source_relTarget

    -- value as defined in initial value
    relTarget: nice_target_table WHERE unique_id = 'identifier'

    -- query that will be executed on the database
    SELECT pk FROM nice_target_table WHERE unique_id = 'identifier'

.. _multi-relations:

multi-relations
^^^^^^^^^^^^^^^

Define queries that should be used to fill n:n relations. The key has to correspond to a relation in Nice. The value
will be used as part of a SQL query to determine the keys that should be inserted in to the relation table. Your value
should contain the table name of the target table and a ``WHERE`` condition that identifies the records you wish to
relate. The information on the relation table name and the foreign key names are taken from the link definition in the
relation file.

.. warning::

    Setting n:n relations will only increase the version on the side that was the source for a change. For example,
    changing Address_relAddress_code1 will only increase the version on the Address, not the Address_code1. Keep this
    in mind when deciding into which initial values file to put your multi-relations, as the wrong side might result
    in all relations being reset on each deployment if the source entity is usually not edited.

.. code-block:: sql
   :caption: Example for a relation Source_relTarget

    -- value as defined in initial value
    relTarget: nice_target_table WHERE unique_id IN ('identifier', 'some-other-id')

    -- queries that will be executed on the database
    SELECT pk FROM nice_target_table WHERE IN ('identifier', 'some-other-id');
    INSERT INTO nice_source_to_target (fk_source, fk_target) VALUES (source-pk, target-pk);

Running initial values from changesets
--------------------------------------

It is possible to run specific initial values from changeset through the use of the
:nice:`YamlInitialValueCustomChange <ch/tocco/nice2/dbrefactoring/impl/data/YamlInitialValueCustomChange>`. See the
class for instructions how to use it, but make sure that you actually need to use it since it is a rather ugly fix for
necessary interactions between existing changesets and new initial values.

Migrating changesets to YAML initial values
-------------------------------------------

There is a action called :nice:`YamlLookupAction <ch/tocco/nice2/dbrefactoring/impl/yaml/YamlLookupAction>` that can
be called directly by a developer. This will find all initial value changesets in the installed modules and will try to
map them to new YAML initial values. It is not super cleanly implemented since it was mainly used to support a manual
migration, so the results need to be checked carefully. But in general, most customers will not need to migrate their
old changesets anyway.

Special logic when upgrading a customer to new initial values
-------------------------------------------------------------

There is a special table ``initial_values_status`` that is used to keep track of the state of the initial values. If
it is the first run of the initial values on this database, and it is not a new database, there is special handling for
some initial value fields when inserting new values:

* any ``active`` flag  is set to false
* these default fields are set to false

  * ``default_income`` and ``default_summary`` on ``Account``
  * ``default_price_category`` on ``Price_category``
  * ``default_payment_condition`` on ``Payment_condition``
  * ``default_schedule`` on ``Payment_schedule``
  * ``default_cost_center`` on ``Cost_center``
  * ``default_vat_code`` on ``Vat_code``
  * ``default_donation`` and ``default_connection`` on ``Esr_account``
  * ``default_currency`` on ``Currency``
  * ``standard`` on ``Voucher_type``
