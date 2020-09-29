App Managagement via Ansible
============================

.. tip::

    This document describes details about Ansible being used to manage
    installations of Tocco. For documentation about managing servers via
    Ansible, have a look at the :ansible-repo-dir:`docs` directory in
    the Ansible repository.

.. tip::

    Setup instructions can be found in :ref:`setup-ansible`.


Implementation Status
---------------------

Implementation status of the automation via Ansible. Unless otherwise noted, this list
concerns application running on OpenShift only.

+---------------+----------------+-----------+----------+----------+---------------------------------------+
| | Category    | | Resource     | | Create/ | | Delete | | Create/| | Notes                               |
|               |                | | Manage  |          | | Manage |                                       |
|               |                |           |          | | (Nine) |                                       |
+===============+================+===========+==========+==========+=======================================+
| DNS           | | A/AAAA/CNAME |           |          |          | No Public API                         |
|               | |  records     |           |          |          |                                       |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| DB (main)     | user           | x¹        | x        | x¹       |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | DB             | x¹        | x        | x¹       |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | app config     | x¹        | n/a      | x¹       |                                       |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| DB (history)  | user           | x¹        | x        | x¹       |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | DB             | x¹        | x        | x¹       |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | schema         | x¹        | n/a      | x¹       |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | app config     | x¹        | n/a      | x¹       |                                       |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| Solr          | user           | ⁵         | ⁵        |          | Account currently shared by **all**   |
|               |                |           |          |          | installations                         |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | core           | x         | x        |          |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | app config     | x         | n/a      |          |                                       |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| Openshift/    | project        | x         | x        |          |                                       |
| Kubernetes    +----------------+-----------+----------+----------+---------------------------------------+
|               | config         | x         | n/a      | x⁴       |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | routes         | x         | x⁶       |          | Including DNS verification            |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | reCaptcha      | x         | n/a      | x        |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | ACME           | x         | n/a      |          | Let's Encrypt integration             |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| TeamCity      | cust. project  | x         | x        |          |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | build config   | x         | x        |          |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | parameters     | x         | n/a      |          |                                       |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| Monitoring    |                | x         | x        |          |                                       |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| S3            | user           | x         | x        | x        |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | bucket         | x         | x        | x        |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | policy         | x         | n/a      | x        | Grant access for developers           |
+               +----------------+-----------+----------+----------+---------------------------------------+
|               | app config     | x         | n/a      | x³       |                                       |
+---------------+----------------+-----------+----------+----------+---------------------------------------+
| Mail          | configure mx   | x         | n/a      | x²       |                                       |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | default sender | x         | n/a      | x²       | Fallback email addressess             |
|               +----------------+-----------+----------+----------+---------------------------------------+
|               | allowed sender | x         | n/a      | x²       | Including SPF and DKIM verification   |
+---------------+----------------+-----------+----------+----------+---------------------------------------+

| ¹ Only managed if ``db_server`` variable is set.
| ² Only managed if ``mail_domains`` variable is set.
| ³ Only managed if ``app_server`` variable is set.
| ⁴ Application properties, DB and S3 setting are managed.
| ⁵ No dedicated user is created. Rather, one user account is shared and configured via Ansible.
| ⁶ Routes managed by Ansible have the annotation 'tocco.ansible-managed' set to 'true. Ansible
|   will not remove any routes that are missing this annotation. Hence, manually created routes
|   as well as dynamically created routes are never removed by Ansible.


Repository
----------

The Ansible configuration is stored in a `Git repository`_ in the ``/tocco`` directory.
The root directory, ``/``, is used for server management.

Overview of the repository structure::

    tocco
        │
        ├── config.yml                                # Definition of existing installations
        │                                             # and parameterization.
        │
        ├── filter_plugins                            # Filter plugins for use with Jinja2.
        │   ├── crypto.py                             #
        │   ├── format.py                             # For instance, in {{ "a_string"|to_camelcase }},
        │   └── parse.py                              # `to_camelcase` is the filter
        │
        ├── inventory.py                              # Script used to parse config.yml and convert
        │                                             # it to a proper Ansible Inventory
        │
        ├── library                                   # Custom Ansible module
        │   ├── backoffice_installation.py            #
        │   ├── cloudscale_s3.py                      # Example showing module defined in teamcity_parameters
        │   ├── teamcity_parameters.py                # being used:
        │   ├── teamcity_project.py                   #
        │   └── vshn_openshift.py                     #     - name: TeamCity - set parameter
        │                                             #       teamcity_parameters:
        │                                             #         user: ansible
        │                                             #         password: '{{ secret }}'
        │                                             #         id: ProjectId
        │                                             #         params:
        │                                             #           branch: master
        │
        ├── playbook.yml                              # Starting point defining which roles
        │                                             # to execute for which installation.
        │
        ├── roles
        │   └── tocco
        │       ├── files                             # Files used in tasks
        │       │   └── history_db.sql                #
        │       │
        │       ├── tasks                             # Instructions for how to setup and configure
        │       │   ├── database.yml                  # installations.
        │       │   ├── mail_domains.yml              #
        │       │   ├── main.yml                      # `main.yml` the starting point and everything
        │       │   ├── route.yml                     # else is included from there as needed.
        │       │   └── teamcity.yml                  #
        │       │
        │       └── templates                         # Templates using the Jinja2 templating
        │           ├── deploymentconfig_nice.yml     # language. This templates are used
        │           └── rolebinding_ansible_edit.yml  # within tasks.
        │
        ├── secrets2.yml                              # Ansible Vault containing passwords
        │                                             # and other secrets in encrypted form.
        │
        └── test_plugins                              # Custom test for use in Jinja2
            └── basics.py                             #
                                                      # For instance, in {{ 5 is even }},
                                                      # `even` is the test.


Configuration (``config.yml``)
------------------------------

Structure
^^^^^^^^^

.. code-block:: yaml

    vars:                                         # Global variables
      db_server: db1.tocco.cust.vshn.net          #
      s3_endpoint: https://objects.cloudscale.ch  #

    definitions:
      abc:                                        # Customer "abc"

        s3_bucket: nice-abc                       # Customer variables for "abc"
        mail_relay: mxout1.tocco.ch               #

        installations:
          abc:                                    # Installation "abc"

            db_name: nice_abc                     # Installation variables for "abc"
            solr_core: nice-abc

          abctest:                                # Installation "abctest"

            db_name: nice_test                    # Installation variables for "abctest"
            solr_core: nice-test                  #


.. _ansible-variable-precedence:

Variable Precedence
^^^^^^^^^^^^^^^^^^^

Variables from highest to lowest priority. Higher priority precedes
lower priority:

* Installation variables
* Customer variables
* Global variables

Example:

.. code-block:: yaml

    vars:
      db_server: db1.tocco.ch
    definitions:
      abc:
        db_server: db2.tocco.ch
        abc:                          # <= db_server is "db3.tocco.ch"
          db_server: db3.tocco.ch
        abctest:                      # <= db_server is "db2.tocco.ch"
      xyz:
        xyz:                          # <= db_server is "db1.tocco.ch"
        xyztest:                      # <= db_server is "db4.tocco.ch"
          db_server: db4.tocco.ch


Merge Variables
^^^^^^^^^^^^^^^

**By default, variables are replaced rather than merged:**

Example:

.. code-block:: yaml

    vars:
      application_properties:
        nice2.request.limit: '1000'
    definitions:
      abc:
        application_properties:
          nice2.history.enabled: 'true'
        abc:
          application_properties:
            nice2.pool_name: 'test'
        abctest:

In the above example, the result will be:

=============== =======================================
 Installation    Resulting Value
=============== =======================================
 abc            .. code-block:: yaml

                    application_properties:
                      nice2.pool_name: 'test'

 abctest         .. code-block:: yaml

                     application_properties:
                       nice2.history.enabled: 'true'
=============== =======================================

**This behavior can be changed using the !merge type:**

.. code-block:: yaml

    vars:
      application_properties:
        nice2.request.limit: '1000'
    definitions:
      abc:
        application_properties: !merge
          nice2.history.enabled: 'true'
        abc:
          application_properties: !merge
            nice2.pool_name: 'test'
            nice2.request.limit: null
        abctest:
            nice2_request.limit: '2000'

In the above example, the result will be:

=============== ===================================================
 Installation    Resulting Value
=============== ===================================================
 abc            .. code-block:: yaml

                    application_properties:
                      nice2.history.enabled: 'true'
                      nice2.pool_name: 'test'

                      # setting the value to null removes the item
                      # nice2.request.limit: null

 abctest         .. code-block:: yaml

                    application_properties:
                      nice2.history.enabled: 'true'
                      nice2_request.limit: '2000'
=============== ===================================================

**Limitations:**

This is only implemented for dictionaries defined directly on the
customer or installation.

**Implementation:**

The ``!merge`` type is implemented within the inventory script (``tocco/inventory.py``). It
handles merging the dictionaries and hands the variables over to Ansible afterwards.

Templating with Jinja2
^^^^^^^^^^^^^^^^^^^^^^

The templating language Jinja2 can be used in variables as well
as on templates and in tasks.

Documentation:

* `Jinja2 Documentation <https://jinja.palletsprojects.com>`__
* `Ansible extensions <https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html>`__

**Example:**

.. code-block:: yaml

    vars:
      db_name: nice_{{ installation_name }}
      history_db_name: '{{ db_name }}_history'
      db_server: |-
        {% if location == 'blue' -%}
        db1.blue.tocco.ch
        {%- else -}
        db1.red.tocco.ch
        {%- endif %}
    definitions:
      abc:                                            # <= db_name is "nice_abc"
        location: red                                 #    db_server is "db1.red.tocco.ch"
                                                      #    history_db_name is "nice_abc_history"

      abctest:                                        # <= db_name is "NICE2_ABCTEST"
        db_name: NICE2_{{ installation_name|upper }}  #    db_server is "db1.blue.tocco.ch"
        location: blue                                #    history_db_name is "NICE2_ABCTEST_history"

**Evaluation:**

Junja2 templates are evaluated for every installation independently. Thus, *{{ installation_name }}*
always correspond to the name of the installation being processed.

Also, all expressions and statements are only evaluated when used. Thus, when setting these variables …:

.. code-block:: yaml

  is_production: "{{ not is_test }}"
  is_test: "{{ installation_name.endswith('test') }}"
  db_user: "{% if location == 'nine' %}{{ installation_name }}_user{% else %}{{ installation_name }}{% endif %}"

… they are **not** evaluated until used. Here for instance by passing them to the debug module:

.. code-block:: yaml

    - name: print debug info
      debug:
        msg: '{{ db_user }}'
      when: is_production

``{{ not is_test }}``, ``{{ installation_name.endswith('test') }}`` and
``{% if location == 'nine' %}…{% endif %}``, defined in the variables above, are only evaluated now,
and will be evaluated again when used again.  Consequently, the variables *installation_name*,
*location* and *is_test* used in the expressions/statements can be referenced before they exist. This
delayed evaluation is used extensively throughout the Ansible playbooks. It allows the use of global,
customer, installation and run time variables without having to worry whether they have been set
at that point.

**Special variables:**

A bunch of special variables are set transparently based on the definitions in ``config.yml`` and can
be used anywhere in a playbook. These variables are set by the inventory script (``inventory.py``).

======================= ========================================================
 customer_name           The customer to which the installation belongs.
 installation_name       Name of the installation.
 sibling_installations   Names of all other installations belonging to the same
                         customer.
======================= ========================================================

Ansible itself has built-in `special variables`_ that can be used too.

Ansible does not understand the concept of customers or installations.  For
Ansible to be able make sense of it, installations are translated to hosts
and customers to groups. This means, for instance, *hostvars*, contains the
variables belonging to all installations and *groups* contains the names
of all customers.

.. hint::

    In Yaml, quotes have to be used for any value starting with ``{{``:

    .. parsed-literal::

      :strike:`db_server:  {{ var }}`       # Invalid, the first { is consider a start of
                                  # dictionary by Yaml.

      db_server: '{{ var }}'      # ok



.. _ansible-app-properties:

Set Application Properties
^^^^^^^^^^^^^^^^^^^^^^^^^^

Application properties can be set via *application_properties* variable:

.. code-block:: yaml

    definitions:
      abc:
        application_properties: !merge
          nice2.history.enabled: 'true'

**Use !merge as described in** `Merge Variables`_.

.. hint::

    The value must be a string. Thus, use quotes where appropriate (i.e. ``'5'``,
    ``'true'``).


.. _ansible-add-route:

Add Routes / Endpoints
^^^^^^^^^^^^^^^^^^^^^^

#. Add the necessary :doc:`DNS entries </devops/openshift/dns>`.

#. Add the route to ``config.yml``:

   .. code-block:: yaml

       definitions:
         abc:   # <- customer
           installations:
             abc:  # <- installation
               routes:
                 abc.ch:
                 www.abc.ch:
                 xyz.ch:              # <= add the new routes here
                 www.xyz.ch:          # <=
             abctest:

   The default route *${INSTALLATION}.tocco.ch* is added implicitly. Only add it
   explicitly if you wish to override the default settings.

   Technical note: the default route, if absent, is added by the inventory script
   (``inventory.py``).

#. Apply change:

    .. parsed-literal::

        ansible-playbook playbook.yml -t route -l **${INSTALLATION}**

.. hint::

   The required TLS certificates are issued automatically in the background.
   This can take some time though. See :ref:`acme-troubleshooting` if you
   run into any issues.


Remove Routes / Endpoints
^^^^^^^^^^^^^^^^^^^^^^^^^

#. Remove route from ``config.yml``

#. Apply change:

   .. parsed-literal::

        ansible-playbook playbook.yml -t route -l **${INSTALLATION}**


Configure Email Sender Domains
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a list of domains that may be used as sender in emails. For instance, if
*tocco.ch* is listed *someuser@tocco.ch* may be used as a sender. Any other sender
address used is rewritten.

#. Ensure SPF and DKIM records are set

   See :doc:`/devops/mail/dns_entries`

#. Set allowed domains

    .. code-block:: yaml

             abc:   # <- customer
               mail_domains:
                 abc.ch:             # <= List domains here
                 abc.net:            # <=
               installations:
                 abc:
                 abctest:

#. Apply change:

    .. parsed-literal::

        ansible-playbook playbook.yml -t mail


.. hint::

    While discouraged, it's possible to set a `mail_domain` without
    adding a SPF or DKIM record by disabled the automated check:

    .. code-block:: yaml

        abc:   # <- customer
          mail_domains:
            abc.ch:
              disable_dkim_check: true    # <= disable DKIM verification
              disable_spf_check: true     # <= disable SPF verification
            abc.net:

    **Expect mails to end up in spam or be refused. Particularly, with
    a missing or incorrect SPF.**



Configure Default Sender Addresses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Set mail addresses

   .. code-block:: yaml

       definitions:
         abc:   # <- customer
           mail_sender_default: info@abc.ch     # <= Address used when sender domain is not listed
                                                #    in `mail_domains` and no default is set on
                                                #    business unit.

           mail_sender_noreply: noreply@abc.ch  # <= Address used in in special context where
           installations:                       #    replying doesn't make sense. For instance,
             abc:                               #    on the password reset mail.
             abctest:

   The domains of the sender addresses **must** be listed in ``mail_domains``. See above.

#. Apply change:

    .. parsed-literal::

        ansible-playbook playbook.yml -t mail -l **${CUSTOMER}**


Configure Monitoring
^^^^^^^^^^^^^^^^^^^^

There are three variables for configuring monitoring:

============================= =======================================================
 monitoring_enabled            | Whether to enabled monitoring.
 monitoring_alert_tocco        | Whether to send alerts to Tocco. As of now, alerts
                               | are mailed to the admin inbox.
 monitoring_alert_vshn         | Whether VSHN should be alerted. (Currently unused)
============================= =======================================================

By default, monitoring is enabled and alerts are sent to Tocco. You can override the
default per customer, installation or route:

.. code-block:: yaml

       definitions:
         abc:
           monitoring_enabled: false  # disable for customer
           installations:
             abc:
               monitoring_enabled: true  # re-enable for installation
               routes:
                 abc.org:
                 www.abc.org:
                   monitoring_alert_tocco: false  # do not send alerts for www.abc.org
             abctest:


Usage
-----

Show Available Installations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code::

    $ cd ${ANSIBLE_GIT_REPO}/tocco
    $ ansible-inventory --graph
      @all:
      |--@tocco_installations:
      |  |--@customer_abbts:
      |  |  |--abbts
      |  |  |--abbtstest
      |  |--@customer_agogis:
      |  |  |--agogis
      |  |  |--agogistest
      |  |--@customer_anavant:
      |  |  |--anavant
      |  |  |--anavanttest
      …

| *abbts*, *abbtstest*, *agogis*, … are installations
| *customer_abbts*, *customer_agogis*, … are customers


Run Full Playbook (=Configure Everything)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. important::

    Always update your repository clone first::

        $ cd ${ANSIBLE_GIT_REPO}/tocco
        $ git pull --rebase

.. code::

    $ cd ${ANSIBLE_GIT_REPO}/tocco
    $ ansible-playbooka playbook.yml -l abbts


.. tip::

    ``-l/--limit`` limits on which installations the playbook is
    executed. You may specify multiple installations and customers
    separated by comma::

        -l abbts,customer_anavant

    This will execute the playbook on installation *abbts* and
    all installations of customer *anavant*.

    Without ``-l/--limit`` the playbook is executed on all installations.


Run Playbook Partially (Tags)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. important::

    Always update your repository clone first::

        $ cd ${ANSIBLE_GIT_REPO}/tocco
        $ git pull --rebase

It's possible to run only parts of the playbook by using what's
called tags. For instance, you can use the tags ``postgres``
and ``s3`` only run tasks setting up these services::

    $ cd ${ANSIBLE_GIT_REPO}/tocco
    $ ansible-playbook playbook.yml -t postgres,s3

Important Tags:

================ =====================================================
 mail             Configure allowed sender domains and default sender
                  addresses.
 postgres         Setup Postgres user and database and configure
                  connection settings in Tocco.
 route            Configure routes including enabling TLS certificates
                  via Let's Encrypt.
 s3               Setup S3 user and bucked and configure it in Tocco.
 teamcity         Setup continuous delivery in TeamCity
================ =====================================================

.. hint::

    A more complete and current list of tags can be obtained via
    ``--list-tags``. To see what tags tasks have assigned use
    ``--list-tasks``.

.. hint::

    ``--skip-tags TAG1,TAG2`` to skip tasks having certain tags assigned.


Run Playbook in Batches
^^^^^^^^^^^^^^^^^^^^^^^

When applying changes to a large number of installations, in particular
**if the change involves an automatic restart**, it's preferable to run the
playbook on a limited number of installations at a time. To this end,
``-e batch=BATCH_DEFINITION`` can be used to run the playbook in batches.

Examples:

Run the playbook for **one installation at a time**::

    -e batch=1

Run playbook on one installation first, then on five, and then keep
running it 20% of the installations::

    -e batch="[1,5,'20%']"

The next batch is started only when all changes could be applied
successfully.

This is internally implemented using Ansible's `serial keyword`_. Any
value accepted by *serial* can be used.


Check Mode
^^^^^^^^^^

The check mode can be used to show what would be changed without actually
applying the changes::

    $ cd ${ANSIBLE_GIT_REPO}/tocco
    $ ansible-playbook playbook.yml --check

.. warning::

    Many of the tasks modifying OpenShift/kubernetes configurations currently
    report incorrectly changes when running in check mode.

    Namely, these tasks currently report changes incorrectly:

    * *create ansible-edit rolebinding / grant TeamCity access for deployments*
    * *create nice deployment config*
    * *set mail domains*


Troubleshooting
^^^^^^^^^^^^^^^

**Debug output**:

Use ``-v`` show parameters passed to a module and the result returned
by it. Use ``-vvv`` to show full debug output.

**Analyze variables**:

You can display variables set for an installation:

.. parsed-literal::

    $ cd ${ANSIBLE_GIT_REPO}/tocco
    $ ansible-inventory --yaml --host **${INSTALLATION}**

or all installations::

    $ cd ${ANSIBLE_GIT_REPO}/tocco
    $ ansible-inventory --yaml --list


Ansible Vault - Passwords and API Tokens
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All passwords are stored in an encrypted Vault. You can
access and edit them via::

    $ cd ${ANSIBLE_GIT_REPO}/tocco
    $ ansible-vault edit secrets2.yml

.. hint::

    You need a password to access it. See :ref:`setup-ansible`.


.. _common.yaml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
.. _Git Repository: https://git.tocco.ch/admin/repos/ansible
.. _serial keyword: https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html#rolling-update-batch-size
.. _special variables: https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html
