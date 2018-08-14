.. _Text-Resources:

Text-Resources
==============

Text-Resources are used to allow localisation of labels. They are stored in ``properties`` files in the ``textresources`` subfolder of the model folder using the naming convention ``language_locale.properties``.

Optional-Module Example:
``optional/modulename/module/model/textresources/language_de.properties``
``optional/modulename/module/model/textresources/language_en.properties``
``optional/modulename/module/model/textresources/language_fr.properties``
``optional/modulename/module/model/textresources/language_it.properties``

Customer-Module Example (Locales ``de`` and ``fr`` installed):
``customer/customername/share/model/textresources/language_de.properties``
``customer/customername/share/model/textresources/language_fr.properties``

.. warning::
   In every ``textresources`` folder, each key must be on the same line in every textresource file available for the respective module.

Example content:

.. code-block:: properties

   entities.Academic_title=Akad. Titel
   client.entities.Academic_title=Akad. Titel
   entities.Academic_title.label=Bezeichnung
   entities.Academic_title.unique_id=Kürzel
   entities.Academic_title.master_data=Stammdaten
   entities.Academic_title.sorting=Sortierung

Accessing Text-Resources
------------------------
There are two ways of accessing Text-Resources. They can either be processed in the backend or directly accessed from the frontend. To allow access to Text-Resources in the client, the prefix ``client.`` needs to be prepended.

Entities, Relations and Forms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In Entities, Relations and Forms Textresources are automatically applied. For each entity, a client and a backend text-resource should be created as followed:

.. code-block:: properties

   client.entities.Entity_name=Entity
   entities.Entity_name=Entity

Each field should be labeled with a single text-resource:

.. code-block:: properties

   entities.Entity_name.field_name=Field

Relations by default show the target-entity text-resource this can be overwritten by setting the the following text-resource:

.. code-block:: properties

   entities.Entity_name.relRelation_name=Relation

If something different is needed, the ``label`` attribute can be used to set any text-resource.

Java
^^^^
To use text-resources in Java, the ``TextResources`` service can be used. It can either return simple text-resources in the current locale or a given locale or format ``TextMessages``. ``TextMessages`` are used for text-resources with placeholders.

.. code-block:: java

   private TextResources textResources;

   //get a simple textresource
   String localizedText = textResources.getText("entities.Entity_name.field");

To add a placeholder, it can be added in square brackets as followed:

.. code-block:: properties

   entities.Room.clientquestion.delete.messagemulti=There are [number] other reservations for the linked event which take place in room [room]. Should the room also be removed from these reservations?

In this message, the number of "other reservations" and the label of the concerned room will be added in the java code that uses this resource.

.. code-block:: java

   private TextResources textResources;

   //create a text message and format it
   TextMessage message = new TextMessage("entities.Room.clientquestion.delete.messagemulti");
   message.setVar("number", "100");
   message.setVar("room", "Example Room");
   String localizedText = textResources.formatTextMessage(message);

Java-Script
^^^^^^^^^^^
The ``getText`` function can be used to directly access text-resources which were prefixed with ``client.`` in java-script.

.. code-block:: javascript

   let localizedEntityName = getText("entities.Entity_name")

Placeholders in javascript are numbered and added in curly brackets.

.. code-block:: properties

   client.dwr.failureMessageWithCode=Your input could not be processed (Error Code: {0}).

In this message the error code can be dynamically set.

To fill in the numbered placeholders, additional parameters can be passed to the ``getText`` function.

.. code-block:: javascript

   let failureMessage = getText("dwr.failureMessageWithCode", errorId)

Freemarker
^^^^^^^^^^
Text-Resources can be loaded using the ``[@loadTextResource path="report.business_unit_dependency.moduleName"/]`` directive.

