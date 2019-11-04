Different Module Types
======================

Nice2 is split into different type of modules.

Core Modules
^^^^^^^^^^^^

Core modules provide the basic functionality of a Nice2 installation. All core modules are always installed on every
installation. A Nice2 installation would not work without them.

Core modules can be found in: ``*``

Optional Modules
^^^^^^^^^^^^^^^^

Optional modules (also called marketing modules) are, as the name lets guess, optional. An optional module represents a
feature or functionality which a customer can add to his installation depending on his demand.

Optional modules can be found in: ``optional/*``

Customer Modules
^^^^^^^^^^^^^^^^

For every customer installation a customer module exists. A customer module configures the installation of the customer.
A customer module defines which optional modules (marketing modules) are installed. Also customer specific adjustments
which only belong to the customer are done in customer modules.

Customer modules can be found in: ``customer/*``

Intermediate Modules
^^^^^^^^^^^^^^^^^^^^

A intermediate module is always needed if a feature or functionality should be added **automatically**
[#f1]_ if two or more specific modules are installed and this new feature should **not be licensed
separately**. If the feature is licensed separately, an optional module must be created instead.

**Example:**
There are two modules ``donation`` and ``sms``. The module ``donation`` has an entity donation and a form ``donation_list.xml``
which displays the donations. The module ``sms`` has an action to send sms. See the following picture.

.. graphviz::

     digraph {
       donation [
         shape=rectangle
         label=<<b>Optional Module: donation</b><br/><br/>Contains te form Donation_list.xml>
       ]
       sms [
         shape=rectangle
         label=<<b>Optional Module: sms</b><br/><br/>Contains the action to send sms>
       ]
       donationsms [
         shape=rectangle
         label=<<b>Intermediate Module: donationsms</b><br/><br/>Extends Donation_list.xml with an sms action>
       ]

       { donation sms } -> donationsms [ label="depends on", dir=back ]
     }

If a customer has installed both modules (sms and donation), the action to send sms should be available
automatically [#f1]_ on the form ``donation_list.xml``. The intermediate module ``donationsms`` which
depends on the modules ``sms`` and ``donation`` adds this action to the form.

The name of an intermediate module consists of the names of the depending modules. For example the module ``licencecorrespondence``
depends on the modules ``licence`` and ``correspondence``.

Intermediate modules can be found in: ``optional/*``

.. hint::
   **A intermediate module**

   * adds functionality automatically [#f1]_ if at least two other specific optional modules are installed
   * depends on at least two other modules
   * is not licensed separately (otherwise it must be an optional module)
   * has a name that consists of the names of the depending modules


.. rubric:: Footnotes

.. [#f1] By *automatically* is meant that the action "Modulabhängigkeiten anzeigen" in :term:`BO` will
         automatically include an intermediate module in the generated tree and pomx.xml if all
         dependencies are satisfied.
