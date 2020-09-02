Coding Guidelines
=================

General
-------

* No code is committed that is not 100% understood by the author.
* A change is always reviewed by another developer. A change is locally tested before it is pushed to Gerrit.
* If multiple independent changes are done they should be separately committed.
* There is an editorconfig to maintain a consistent code layout. It defines the file encoding, linebreaks, indentation and trailing whitespaces.
* The file encoding is always UTF-8.
* Code and comments should be always in American English.
* Do not comment out unused code. The version control system tracks the changes.

Commit Message
--------------

* On the first line is the Gerrit subject. There is a limit of 72 characters. The message is written in the present (E.g. *change* and not *changed* or *changes*)
* If more details are helpful add a message body. Between the subject and message body must be an empty line.
* Use the Refs or Closes command to link your commit to a Jira task.
* The Gerrit hook generates a change-id which is part of the commit message. Do not modify this id.
* Here a full example:

.. code-block:: bash

   add new monitoring module

   - add logic to track requests
   - add visualization of requests over time
   - add custom JSON exporter

   Refs: TOCDEV-1234
   Change-Id: I35106653d15655c1b7d988ff0062da9fc9e41893

Java
----

General
^^^^^^^

* Always include curly brackets, even for one-line if statements or for loops.
* One statement per line. Avoid too long line (make a line break after ~ 120 characters).
* Avoid the if ternary operator.
* Each interface has a corresponding unit test named *InterfaceName* Test.java
* Strings are concatenated with ``String.format(...)`` and not ``"String1" + "String2"``.
* Create lists and sets which are always empty with ``List.of()`` and ``Set.of()``. Else use the Guava library to create lists and sets  (E.g. ``ImmutableList.of(1, 2)``).
* Each mock in a test must be verified.
* A class has the following structure (sorting of the different class elements):

   * Variables:

      * Constants (``static final``)
      * Injected services
      * Other variables such as mocks in tests

   * Constructors
   * Methods: First public, then protected, and then private methods are defined

Naming
^^^^^^

* Interfaces have no *I*-prefix. If an interface has only one implementation this class has the suffix *Impl* (E.g. Interface: ``PizzaFactory`` and Implementation ``PizzaFactoryImpl``).
* Methods and field names are declared in camel case (E.g. ``getCmsPage()``).
* Constants are declared in uppercase separated by underscores (E.g. ``private static final int CACHE_SIZE = 100;``).
* Class members are always private and getter/setters are used for interaction.

Parameters and Return Values
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* The ``@Nullable`` annotation from the ``org.jetbrains.annotations`` package is used. But parameters and return values are not annotated with ``@NotNull``.
* An alternative to a nullable return value is an optional from Java (not Guava).
* Parameters are validated if the input is in the expected range. Else an ``IllegalArgumentException`` is thrown.

Exception Handling
^^^^^^^^^^^^^^^^^^

* Catch blocks are never empty. At least log the error.
* Use the logger and not ``System.{out, err, ...}`` or ``Exception#printStrackTrace()``.
* Before an exception is thrown the reason should be logged.
* A method should throw only checked exceptions which are relevant for the caller. Less relevant exceptions, which you do not want to handle yourself, should be wrapped in a general exception.

JavaDocs
^^^^^^^^

* All classes and all methods which are not private have JavaDoc. (Exceptions are getters & setters and standard Java beans). The documentation describes the purpose of the class and points out special cases etc.
* Information about the author (``@author``) and the creation time (``@since``) is not relevant in JavaDoc - for this we have a VCS.
* JavaDoc for methods (except private ones) contains at least (if useful) ``@throws``, ``@returns``, ``@param``.
* Use the markdown syntax for formatting and not HTML.

ExtJS
-----

General
^^^^^^^

* Always include curly brackets, even for one-line if statements or for loops.
* One statement per line. Avoid too long line (make a line break after ~ 120 characters).
* Do comparison operators always with the type-safe operator ``===`` and ``!==``.
* Do not use ``Console.{log, error, ...}``.
* Usually do no custom error handling, because in case of an error a standard message is displayed.
* If something did not work on the server, then throw an exception. Only then the failure method can be called on the client (and the user can be informed accordingly). If further information is needed in case of an error, then implement an own exception, which implements the ``ch.tocco.nice2.netui.impl.dwr.InformationException``. This allows to access its information in the failure method.
* In case of an error, keep the latest version if possible (e.g. do not close an open window).

Naming
^^^^^^

* Methods and field names are declared in a camel case (E.g. ``getCmsPage()``).
* If a scope variable is needed, it is named ``me`` ( → ``var me = this;`` and not that, self, ...). In general, you should not use scope variables and execute a function directly in the correct scope (E.g. ``myFunc.call(scope);``).
* If an action was successful the user should get a success message.
* If an error occurs the user should always get an error message.

XML
---

* Use hyphen syntax for element and attribute names, i.e., all lowercase letters; do not use camel case for elements (E. g. ``set-status-date``).
* If a publicform is customer specific changed it cannot be partially replaced. It requires:

.. code-block:: XML

   <form xmlns="http://nice2.tocco.ch/schema/formModel.xsd" data="My_entity" replace="true">

* Non trivial entities, relations and fields are documented with the ``<documentation>`` element.


