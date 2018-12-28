Forms
=============

Entities can be displayed in various views. Forms definitions determine how these views are arranged.

There is an old and a newer form-model. The later was introduced to fit the new client's needs. This chapter describes attributes
and tags that are valid for the new form-model. Most of them will also work for the old one though.
Those that are exclusively for the new form model are marked with a :green:`V2`


Types and naming 
----------------

As of today there are five different types of forms:

==================== ================================================================
Type                 Description
==================== ================================================================
list                 To display multiple entities of the same kind in a list. 
detail               To display one entity. This can either be a read-only or writable views.
create               Special view to create an entity. If not defined, detail view is used as fallback.
update               Special view to update an entity. If not defined, detail view is used as fallback.
search               Optional search form that gets embedded above a list form.
==================== ================================================================

The forms are defined as an xml file in to following folders:

    * File-Path in an optional module: ``optional/{modulename}/module/model/forms`` 
    * File-Path in a customer module: ``customer/{customername}/share/model/forms``


A form file has following naming schema: ``{Name, PascalCase}_{Type}.xml``

To create an entities default view, *{Name}* is substituted with the entity name. Entity names are written as they are and not in PascalCase.
If a form is created for a flow *{Name}* is usually substituted with the flows name.

Examples: ``Event_category_detail.xml``, ``RegistrationAdministration_list.xml``

Auto Forms
^^^^^^^^^^
If there is no xml definition for an entity form a list, detail and search form is generated automatically.
This happens during the runtime and the form will not be persisted in a xml file.
The generated form simply contains all fields of the entity.

List Forms
-----------
Simple Example:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <form xmlns="http://nice2.tocco.ch/schema/formModel.xsd">
      <table data="User">
        <field data="firstname"/>
        <field data="lastname"/>
        <column sortable="false">
          <field data="email"/>
        </column>
      </table>
    </form> 


<Form>
^^^^^^^^
data (String)
+++++++++++++
Name of the entity that should be listed.


<Table>
^^^^^^^^^

constriction (String)
+++++++++++++++++++++
Listed entities are filtered with this constriction. (A constriction is like a filter but is defined within a hivemodule.xml)

sorting (String)
++++++++++++++++
List of fields separated by comma. This sorts the records in ascending order by default. If a descending order is needed
a minus sign (-) can be prepended to the field name.

Example:
``sorting="firstname,-lastname,relCountry.name"``


If not specified the entities default sorting will be applied.

selectable (*true*, false) 
++++++++++++++++++++++++++
If true, checkboxes are shown on each row and one or multiple records can be selected. 
This feature is needed if there are actions or reports that should be triggered for selected records.

<field>
^^^^^^^
.. _data-anchor:

data (String)
+++++++++++++
Path to field. Fields related with a n to 1 relation can also be visualized using dots and the relations name.

``data="relUser_user2.relRegistration.relEducation_requirement_status"``

<Column>
^^^^^^^^^^^^^^
A Column is used as a wrapper to add special attributes.

name
++++++
The name is used to retrieve the corresponding text-resource or reference a position.

.. _display-type-anchor:

display-type (String; *editable*, readonly, hidden)
++++++++++++++++++++++++++++++++++++++++++++++++++++
Determines how the field is displayed. If hidden, the label is also not shown.
At the moment, *editable* is not regarded in the context of list views.

sortable (*true*, false)
++++++++++++++++++++++++
Whether there is an arrow shown in the column that let the user sort the table accordingly.


Detail, Search, Create & Update Forms
--------------------------------------

Simple Example:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <form xmlns="http://nice2.tocco.ch/schema/formModel.xsd">

    <horizontal-box labeled="false">
      <vertical-box labeled="false">
        <vertical-box name="user_information" labeled="true">
          <field data="firstname"/>
          <field data="lastname"/>
          <field data="email"/>
        </vertical-box>
        <vertical-box name="marketing_information" labeled="true">
          <field data="relAffiliation" scopes="update,read"/>
        </vertical-box>
      </vertical-box>
    </horizontal-box>

    </form>

<horizontal-box>, <vertical-box>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Boxes for layouting purposes.
Search forms do not support any layouting boxes.

labeled (true, false)
+++++++++++++++++++++++
If a layouting box is labeled the name tag is required. 
The textresource is getting resolved with the following path: ``forms.{form_name}.{boxes}.{name}``
{boxes} is substituted with all named boxes on a higher level (eg. box1.box1_subbox...)

label (String)
++++++++++++++
The default text resource path can be overwritten with a custom path.

name (String)
++++++++++++++
Only needed if labeled is *true* and label is not set.

<field>
^^^^^^^^
data (String)
+++++++++++++
Same as in list form :ref:`data-anchor`


display-type
+++++++++++++
Same as the display-type used for a column. See :ref:`display-type-anchor`

use-label (String, *yes*, no, hide)
++++++++++++++++++++++++++++++++++++
If *yes* a special form text-resources is looked up:
``forms.{formName}.{boxes}.{fieldName}``

If not defined the entities field text-resource is used.

For more information about text-resources see :ref:`Text-Resources`

label (String)
++++++++++++++
Overwrite the default text-resource path.

scopes (String; read, update, create)
+++++++++++++++++++++++++++++++++++++

Selectively show field. For example if the scope of a field is set to update, the create view will not show it.

``scopes="update,read"``

<display>
^^^^^^^^^
Is used to embed freemarker expressions into the form.

.. code-block:: xml

    <display name="relDebitor.balance_open" language="freemarker" escape-html="false">
      <![CDATA[
          <h1>Title</h1>
      ]]>
    </display>

The `escape-html` attribute allows you to define if html is escaped in the resulting text. The `compressible` attribute
allows you to define if whitespace should be compressed in the resulting text.

<table> :green:`V2` 
^^^^^^^^^^^^^^^^^^^
A table in a detail form is called a **sub-table**. Its used to list to n relations.


.. code-block:: xml

    <horizontal-box labeled="false">
      <table name="relIncoming_payment" data="relIncoming_payment" show-search-form="false" limit="25"/>
    </horizontal-box>

data (String)
++++++++++++++
Name of the relation that should be shown in the sub-table.

name (String)
+++++++++++++
Is used to set an id on the sub-table. If no name is set, the id is *null*.

endpoint (String) 
+++++++++++++++++++
By default, the list view retrieves its data from the default entities endpoint. With this attribute its possible to overwrite
this default behavior.

If used in a sub-table, the ``{parentKey}`` placeholder can be used.

Example:
``endpoint="entities/Requirement/{parentKey}/entitydocs"``

GET http method is used to call the endpoint.

show-search-form (true, *false*)
++++++++++++++++++++++++++++++++++
Whether a search form is displayed or not.

limit (Int) 
++++++++++++++++
How many records to show on one page. If not defined usually the REST default of 25 is shown.


Form composition
-----------------
A form can be changed by any module that extends the module containing the form. Special attributes help to position additional components.
The order of the composition is depending on the module dependencies. For more information see :ref:`Modules`


Example that adds the field *relConstricted_business_unit* to the *Event_category_detail* form

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <form xmlns="http://nice2.tocco.ch/schema/formModel.xsd">

    <horizontal-box name="box1" labeled="false">
      <vertical-box name="box1" labeled="false">
        <vertical-box name="master_data" labeled="true">
          <field data="relConstricted_business_unit" position="after:sorting"/>
        </vertical-box>
      </vertical-box>
    </horizontal-box>

    </form>


position (String; top,bottom,*middle*,before,after)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The position of a component can be applied to <field> or <column>.

*before* and *after* need a relative position such as a field name appended.

``position="after:firstname"``
``position="before:firstname"``

remove
++++++

To remove components, a special file with the following naming convention has to be created: ``remove{formName}.xml```

Example:

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
    <remove xmlns="http://nice2.tocco.ch/schema/removeForm.xsd">
    <form name="User_detail">
      <component path="box1.box2.employee_information.relEmployment"/>
    </form>
  </remove>



Actions
-------
TODO