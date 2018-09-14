.. _Entities and Relations:

Entities and Relations
======================

Entities and Relations define the database model of an application. For each entity, a table will be created. Relations are represented as either foreign keys (**n:1**, **n:0..1**) or connectivity tables (**n:n**).


Entities
--------

The entity configuration consists of global `Entity-Attributes`_, a `Field-Configuration`_ and a `Visualisation`_-Configuration. This configuration is written in xml and must be stored in a file using the naming convention ``Entity_name.xml`` in the ``entities`` subfolder of the model folder.

Example-Path in an optional module: ``optional/modulename/module/model/entities/Entity_name.xml``
Example-Path in a customer module: ``customer/customername/share/model/entities/Entity_name.xml``

.. code-block:: xml

   <?xml version="1.0" encoding="UTF-8"?>
   <entity-model
       xmlns="http://nice2.tocco.ch/schema/entityModel.xsd"
       index-priority="-1"
       remote-field-new-button="false">

     <key name="pk" type="serial"/>

     <field name="unique_id" type="identifier">
       <validations>
         <mandatory/>
       </validations>
     </field>
     <field name="label" type="string" localized="true">
       <validations>
         <mandatory/>
       </validations>
     </field>
     <field name="default_email_sender" type="email"/>
     <field name="noreply_email_sender" type="email"/>
     <field name="report_header_vertical" type="document" localized="true"/>
     <field name="sorting" type="sorting"/>

     <visualisation>
       <sorting>sorting, label</sorting>
       <display language="freemarker" expression="${baseData.label}"/>
     </visualisation>

   </entity-model>


Entity-Attributes
-----------------
Entity-Attributes are attributes of the ``<entity-model>`` tag.

.. code-block:: xml

   <entity-model
       xmlns="http://nice2.tocco.ch/schema/entityModel.xsd"
       index-priority="-1"
       remote-field-new-button="false">

index-priority (int)
^^^^^^^^^^^^^^^^^^^^
Used for fulltext search (index)

* higher value means higher priority

* default for lookup-entities: 0

* default for "normal" entities: 1

* exclude it from index: -1


remote-field-new-button (*true*, false)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Defines whether remotefields with the target equals this entity have the create button displayed per default or not.

session-only (true, *false*)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Defines whether this entity should be used as session-only entity or not. Session only entities can not be saved to the database and are for example used for Report Settings.

markable (true, *false*)
^^^^^^^^^^^^^^^^^^^^^^^^
Defines whether the marking-stars may be used for this entity or not.

relevant-for-export (true, *false*)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Information for FormModelAction, only main entities should be marked as relevant for export

output-center (true, *false*)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Set to true if a relation to output_job_item is required.

widget (main, child)
^^^^^^^^^^^^^^^^^^^^
This is used to mark widget configuration entities. This is required to properly track the publish status.

* *main* is used to mark a normal conf entity (e.g. Collapse_conf)

* *child* is used to mark sub-configurations (e.g. Collapse_item). Updates of a child will amend the publish status of the page where the related *main* entity.


content-reference (*true*, false)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Set to false if this entity is not a content reference source even tough it owns HTML fields.

entity-docs (*none*, multi_language, single_language)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Defines whether this entity will have a document tab or not. ``single_language`` is the only option that is currently in use.

business-unit (*none*, single, manual_set, optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Defines how this entity handles Business-Units

* *none*: Visible in all Business-Units. No relation to ``Business_unit`` will be created.

* *single*: Only visible in the Business-Unit it was created in. A **n:1** relation to ``Business_unit`` will be created.

* *manual_set*: Only visible in the Business-Unit that was selected. A **n:1** relation to ``Business_unit`` will be created.

* *optional*: If a Business-Unit is set, only visible the selected Business-Unit. If nothing is set, every Business-Unit will see this entity. A **n:0..1** relation to ``Business_unit`` will be created.


type (*standard*, lookup)
^^^^^^^^^^^^^^^^^^^^^^^^^
Differences between standard and lookup entities

* There is no entity history for lookup entities

* Relations to lookup entities will be displayed in simple Comboboxes in an Edit-Form. Relations to standard entities will be displayed as Remote-Fields.

Field-Configuration
-------------------
The field configuration contains as well information used to create database columns as information used to initialize the corresponding frontend components.

.. code-block:: xml

   <field name="label" type="string" localized="true" xss-filter="false"/>

Attributes
^^^^^^^^^^

* name (string)
   The name of the entity-field. This name has to be unique for this entity-model. If this name is specified in more than one configuration, the system will merge them to a single entity-field.

* label (string)
   The label of this field. If not specified it will be a textresource whith the key following the rule ``entities.EntityName.fieldName``

* localized (true, *false*)
   If the field is localized, a field for each locale will be created. These fields will be called ``name_locale`` (e.g. ``label_de``).

* doc (string)
   Documentation of the entity, describing its meaning, purpose and important information to mention regarding the usage of the model. The rendering process of the entity model's documentation merges documentation into extending models and supports Markdown syntax. Therefore, the usage of Markdown is recommended to highlight important information or so tag specific terms as code elements. supports Markdown syntax.

* xss-filter (*true*, false)
   XSS-filter activates XssProtectionFieldValidator. Activated by default on any string-based field.

* unique (true, *false*)
   Adds a validator that checks a field value for uniqueness.

* type (string)
   See `Field-Types`_.

* target (string)
   The target of this field. For example the database-table-field. If this isn't specified, it will be the same as the name of the field.


Field-Types
^^^^^^^^^^^

* binary
   Field to store binaries. It will be stored as nullable ``VARCHAR(40)`` in a postgres db. Binaries are stored as large objects in the postgres database and there is a reference table called ``_nice_binary`` which contains the hash and the large object id for each binary. The ``binary`` field, stores the 40 character hash of the binary.

* birthdate
   Date field that is used to store birthdays. It will be stored as ``DATE`` in a postgres db.

* boolean
   Field to store boolean values. In an edit-form this will be displayed as checkbox. It will be stored as not null ``BOOLEAN``  in a postgres db.

* counter
   Long field that is automatically set to the next available number in the current business unit. This uses the ``Counter`` entity to track the current counter value of each business-unit. It will be stored as ``BIGINT``  in a postgres db.

* date
   Field to store dates. In an edit-form this will be displayed as Datepicker. It will be stored as ``DATE`` in a postgres db.

* datetime
   Field to store datetimes. In an edit-form this will be displayed as Datetimepicker. It will be stored as ``TIMESTAMP`` in a postgres db.

* document
   Field to store documents. It will be stored as nullable ``VARCHAR(40)`` in a postgres db.

* email
   Field to store e-mail addresses. The content will be validated and needs to be a "real" e-mail address. It will be stored as nullable ``VARCHAR(255)`` in a postgres db.

* phone
   Field to store phone-numbers. The content will be validated using our phone number library. It will be stored as nullable ``VARCHAR(255)`` in a postgres db.

* serial
   Long value that will be incremented automatically on the database. This is used for primary keys. It will be stored as ``BIGINT`` in a postgres db.

* string
   String field for short texts (e.g. firstname, fastname, label, ...). These fields will be displayed as textfields. It will be stored as ``VARCHAR(255)`` in a postgres db.

* text
   String field for long texts (e.g. description, ...). These fields will be displayed as text-areas. It will be stored as ``TEXT`` in a postgres db.

.. _Entity-Validation:

Validation
----------
Validations can be defined for fields and relations.

.. code-block:: xml

   <field name="num_ratings" type="integer">
     <validations>
       <number-range from-including="0"/>
       <mandatory/>
     </validations>
   </field>

Available-Validations
^^^^^^^^^^^^^^^^^^^^^
* mandatory
   works without arguments.

.. code-block:: xml

   <mandatory/>

* blank
   not in use
* length
   only strings with a length ``from-including`` to ``to-including`` will be accepted.

.. code-block:: xml

   <length from-including="6" to-including="8"/>

* number-range
   only numbers ``from-including`` to ``to-including`` will be accepted.

.. code-block:: xml

   <number-range from-including="0" to-including="100"/>

* regex
   can be used to validate a field using a regular expression.

.. code-block:: xml

   <regex continue="false" level="ERROR" name="regex1">^.*@.*\.[a-zA-Z]{2,5}$</regex>

* decimal-digits
   allows restrictions to the number of ``pre-point`` and ``post-point`` digits.

.. code-block:: xml

   <decimal-digits post-point="2" pre-point="12"/>

* iban
   works without arguments.

.. code-block:: xml

   <iban/>

.. _Default-Values:

Default-Values
--------------
Default Values may be set on field, relation or form level.

.. code-block:: xml

   <field name="creation_date" type="datetime">
       <default set-on-template="if_empty">today</default>
   </field>

type
^^^^
The default for this attribute is hard.

* hard
   Hardcoded content. This will only work for fields.

.. code-block:: xml

   <default type="hard">true</default>

* textresource
   Set a textresource as content (needed for language specific). This will only work for fields.

.. code-block:: xml

   <default type="textresource">reports.Grade_table.label</default>

* query
   To handle the content as a query. This will only work for relations.

.. code-block:: xml

   <default type="query">unique_id == "open"</default>

* querysingle
   Selects the only element for a relation. This will only work for relations.

.. code-block:: xml

   <default type="querysingle"/>

* freemarkerquery
   use freemarker to obtain defauft value. This will only work for relations.

.. code-block:: xml

   <default type="freemarkerquery">
     [@query name="currencyList"]find Currency order by sorting[/@query]
     unique_id == "[@loadValue entity=currencyList?first path="unique_id"/]"
   </default>

* template
   To handle the content as a template and then as a query. This is currently not used.

* null
   Remove a default value.

set-on-template
^^^^^^^^^^^^^^^
The default value for this attribute is ``no`` if the field is writable. On readonly fields, the default gets always set if the field is empty.

* no
   Don't set the default value.

* if_empty
   Only set the default value if the field is empty.

* force
   Set the default value regardless of the existence of a value.

Visualisation
-------------
sorting
^^^^^^^
Specify the default-ordering for this entity-model. Use a comma-seperated list of the fields.
The first field has the highest priority, the last the lowest.
With a dash (minus) before the field-name, the field is descending ordered instead of ascending.

.. code-block:: xml

   <visualisation>
     <sorting>sorting,-last_post</sorting>
   </visualisation>


display
^^^^^^^
Defines a display for an entity.

.. code-block:: xml

   <visualisation>
     <display language="freemarker" expression="${baseData.label}"/>
   </visualisation>

Attributes of the diplay type:

* **expression** (string)
   Optional. This attribute can be used to define the Expression to be displayed. Alternatively, the element content can be used.

   Example of a tooltip that does not use the expression tag.

.. code-block:: xml

   <display type="tooltip" language="freemarker">
     <![CDATA[<b>[@loadTextResource path="entities.Address.address_nr"/]: ${baseData.address_nr?c}</b>
       <table style="border: none; text-align: left; width: 100%;" border="0">
         <tbody>
           <tr>
             <td align="left" valign="top">[@loadTextResource path="entities.Address.tooltip.type"/]:</td>
             <td align="left" valign="top">${baseData.relAddress_type.label}</td>
           </tr>
         </tbody>
       </table>
     ]]>
   </display>

* **language** (*javascript*, freemarker)
   Defines which language is used to process the expression. Only freemarker should be used. ``javascript`` is **deprecated**.

* **type** (string)
   Optional. If this is not set, the display will be used as default display. If an entity needs more than one display,
   they can be distinguished using the type attribute. Common types are ``tooltip`` (used in popup tooltips, ``search``
   (Used for fulltext search results) and ``resourceCalendarTooltip`` (used in Ressource-Calendar).

   Default displays are used to give a string representation of an entity to the user. They appear in a variety of
   places, for instance as the display value of remote fields or the list of opened entities in the list.

   .. figure:: resources/default_display.png
    :alt: a default display in the list of open entities

    Default display of an ``Address`` entity as evaluated in the list of open entities.

Relations
---------
The relation configuration consists of a `Source- & Target-Elements`_ configuration, the `Cardinality`_ and additional `Optional Configuration`_. This configuration is written in xml and must be stored in a file using the naming convention ``Entity_name_relRelation_target.xml`` in the ``relations`` subfolder of the model folder.

.. tip::

   In most cases the ``Relation_target`` is the entity name of the target entity.

Example-Path in an optional module: ``optional/modulename/module/model/relations/Entity_name_relRelation_target.xml``

Example-Path in a customer module: ``customer/customername/share/model/relations/Entity_name_relRelation_target.xml``


.. code-block:: xml

   <?xml version="1.0" encoding="UTF-8"?>
   <relation xmlns="http://nice2.tocco.ch/schema/relation.xsd">
     <source entity-model="User">
       <delete cascade="no"/>
       <display show="false"/>
     </source>
     <target entity-model="User_code1"/>
     <cardinality>n:n</cardinality>
   </relation>

Source- & Target-Elements
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: xml

   <source entity-model="Membership" name="relSingle_user">
     <delete cascade="no"/>
     <display show="false"/>
   </source>
   <target entity-model="User" name="relSingle_user_membership" label="">
     <delete cascade="no"/>
     <display show="true" order="30"/>
   </target>
   <cardinality>n:0..1</cardinality>

Source- and Target-Elements define how the relation works. The source as well as the target element consists of attributes and child elements.

**Attributes:**

* entity-model
   The entity-model-name this name refers to the names used in the entity-configurations.

* name
   The relation-name. If not specified, it will be ``relEntity_name``. The source/target-entity and the name of the relation identifies it. If you specify a relation on two or more locations, the configurations-system will merge it to a single relation.

   .. tip::
      If multiple relations to the same target entity are needed (e.g. a "target" User and an "apply" User on Licences), they have to be named accordingly.

* label
   A label for this relation. If nothing specified, it will be ``entities.{target/source-entity-name}.{relation-name}``.

* lock
   Does this relation lock the related entities if the source-entity is locked? Be careful to not cascade-lock the entire database. **When in doubt, leave it out.**

**Child-Elements:**

* delete
   Specify the delete-behavior of this relation with the ``cascade`` and ``visualisation`` -attribute.  ``cascade`` defines wheter there will be cascading deletion or not, ``visualisation`` specifies how the cascading-deletion is presented to the user. Visualisation is mandatory if cascading delete is used and forbidden otherwise.

   **cascade-options:**

   * ``no``: no cascading delete (default)

   * ``if_last``: only cascade-delete, if no other entity of this type refers to the same instance as the current entity does

   * ``yes``: cascade-delete

   * ``deny``: deny cascading delete

   **visualisation-options:**

   * ``silent``: the user doesn't see the cascade-deleted items

   * ``warn``: the user sees the cascade-deleted items, but he cannot change it.

   * ``editable``: the user sees the cascade-deleted items and can select and deselect it.

* display
   Specify wheter a tab of this relation should be shown on the respective entity. Display is configurable with the two attributes ``show`` (boolean) and ``order`` (int). ``show`` defines whether the relation tab will be shown or not. If it is shown, the position of the tab can be controlled with ``order`` (lower ``order`` tabs are further to the left).

* selector
   Selectors allow to select single entity when resolving a n:n or 1:n relation. This selected entity can then be used in forms. For instance, you could create selector 'current' which selects the current license and then reference it in a form like this: ``<field data="relLicense[current].valid_until">``

.. code-block:: xml

   <target entity-model="User" name="relSingle_user_membership">
     <selector name="new" type="script:js">
       <![CDATA[
         function isSelected(model, source, target) {
           return source.getKey() == null; // always create a new membership
         }
       ]]>
     </selector>
   </target>

* visualisation
   The visualisation element can be used to change how a relation is displayed (e.g. Radio buttons instead of a Combobox)

.. code-block:: xml

   <source entity-model="User">
     <delete cascade="no"/>
     <visualisation>
       <select-box>
         <layout>
           <radio-group orientation="vertical" num-columns="1"/>
         </layout>
       </select-box>
     </visualisation>
   </source>

Cardinality
^^^^^^^^^^^
Available cardinalities are ``n:0..1`` (optional), ``n:1`` (mandatory) and ``n:n``.

Optional Configuration
^^^^^^^^^^^^^^^^^^^^^^

**Attributes:**

* replace (true, *false*)
   When true then this relation model replaces an already existing one. If false (default) then the relation is new or extends an existing relation, and may be overridden later.

   .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8"?>
      <relation xmlns="http://nice2.tocco.ch/schema/relation.xsd" replace="true">
        ...
      </relation>


* disabled (true, *false*)
   If this relation shall be ignored after loading. The syntax still has to validate, but after that it's as if it would not exist. The name of the entity may be the same as one of another file (no uniqueness enforced). This is useful for example when making a new version of an entity but not activating it yet.

   .. warning::
      This should probably never be used.


**Elements:**

* link
   How do the source and target get linked together. This is very backend-specific.

   JDBC:

   * on n:n ``link_table:source_keyfields:target_keyfields``

   .. code-block:: xml

      <link>nice_event_to_address:fk_event:fk_address</link>

   * on n:1 ``source_keyfields``

   .. code-block:: xml

      <link>fk_client</link>

   * On multiple-keyfields, the field are comma-seperated. The system first tries to take the entity-names. If it fails, it takes the backend-names.

   .. warning::
      The link configuration should only be configured if it really needs to deviate from the default configuration!

* default
   Defines the default value for this relation. See `Default-Values`_.

* validations
   Can be used to make a ``n:n`` relation mandatory. See `Validation`_. **Most validation options won't work for relations.**

   .. warning::
      only use this for ``n:n`` relations. ``n:0..1`` should be changed to ``n:1`` if it needs to be mandatory.
