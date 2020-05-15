.. _generated-values:

Automatically generated values
==============================

For data types whose values should be created or updated automatically a :abbr:`FieldGenerator (ch.tocco.nice2.persist.hibernate.pojo.FieldGenerator)`
can be contributed.
The value of these fields are automatically created when a entity is inserted and/or updated.
The :abbr:`FieldGenerator (ch.tocco.nice2.persist.hibernate.pojo.FieldGenerator)` returns a :java-hibernate:`GenerationTiming <org/hibernate/tuple/GenerationTiming>` (defines if the value should
be generated when the entity is created or when it is updated) and a supplier which generates the value.

.. note:

    For example the :abbr:`CreationDateTimeFieldContribution (ch.tocco.nice2.userbase.types.CreationDateTimeFieldContribution)`
    is registered for the data type ``createts`` and creates a timestamp when a new entity is created.

The field generators are stored in the :ref:`classLoaderService` so that they can be accessed through the session later on.

If a field generator exists for a given data type, the :abbr:`AlwaysGeneratedValue (ch.tocco.nice2.persist.hibernate.pojo.generator.AlwaysGeneratedValue)`
annotation is added to this field (or the :abbr:`InsertGeneratedValue (ch.tocco.nice2.persist.hibernate.pojo.generator.InsertGeneratedValue)`
if the value should only be generated once when the entity is first created). Both annotations take the data type as a mandatory parameter (see :ref:`generated-fields-annotations`).

These annotations are themselves annotated with Hibernate's :java-hibernate:`ValueGenerationType <org/hibernate/annotations/ValueGenerationType>`
annotation (see `documentation <https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#mapping-generated-ValueGenerationType>`_).
The :java-hibernate:`ValueGenerationType <org/hibernate/annotations/ValueGenerationType>` annotations refer to an implementation of :abbr:`AbstractFieldGeneration (ch.tocco.nice2.persist.hibernate.pojo.generator.AbstractFieldGeneration)`
which looks up the correct :abbr:`FieldGenerator (ch.tocco.nice2.persist.hibernate.pojo.FieldGenerator)` in the session
(depending on the data type) and generates the value.