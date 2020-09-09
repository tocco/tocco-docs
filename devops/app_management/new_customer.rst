Create new Customer or Installation
===================================

Add DNS Entry
-------------

Usually, during initial setup, installations are only made availiable at
*https\//${INSTALLATION}.tocco.ch* for which a DNS record can be added as
described in :ref:`dns-managed-by-us`.

Should you need an additional, non-\ *tocco.ch* records, have a look
at :doc:`/devops/openshift/dns`.


Create OpenShift Project, S3 Bucket and Database
------------------------------------------------

.. warning::

     If you haven't setup Ansible yet, now is time to follow
     the instructions in :ref:`setup-ansible` and read through
     :doc:`ansible`.

#. Update your `Ansible Repository`_ clone::

       cd ${PATH_TO_ANSIBLE_REPOSITORY}/tocco     # Note the `/tocco`
       git pull --rebase

#. Add customer/installation to ``tocco/config.yml``

    .. code-block:: yaml

       definitions:
         # ...
         abc:                                            # <-- customer
           mail_sender_default: info@domain.ch           # <-- fallback mail address
           mail_sender_noreply: noreply@domain.ch        # <-- fallback noreply address
           mail_domains:                                 # <-- domains allowed as email sender address
             domain.ch:
             domain.net:
           installations:
             abc:                                        # <-- installation
               db_server: db1.tocco.cust.vshn.net
               solr_server: solr2.tocco.cust.vshn.net
             abctest:                                    # <-- installation
               db_server: db1.tocco.cust.vshn.net
               solr_server: solr2.tocco.cust.vshn.net
         # ...

   Just as shown in the example above, **do** use *db1.tocco.cust.vshn.net* and
   *solr2.tocco.cust.vshn.net* as DB and Solr server respectively. This example
   will be updates should other servers be used in the future.

   .. important::

       Naming conventions:

       ===================== ===================================================
        Customer name         May only contain lower-case letters a-z and
                              digits 0-9.

        Installation name     * May only contain lower-case letters a-z,
                                digits 0-9 and hypens (-).
                              * All installation names should start with
                                with the customer name.
                              * The **production** system **must** have the
                                same name as the customer itself. [#f1]_
                              * The **primary test** system **must** be called
                                *{{ customer_name }}test*.
       ===================== ===================================================

   .. hint::

          More details about Ansible is available in :doc:`ansible`

          Should you need more routes, see :ref:`ansible-add-route`.

#. Run Ansible Playbook

   Run playbook for installation **abc** and **abctest**:

   .. parsed-literal::

          cd ${GIT_ROOT}/tocco
          ansible-playbook playbook.yml -l **abc,abctest**

   Or run it for all installations belonging to **customer abc**:

   .. parsed-literal::

          cd ${GIT_ROOT}/tocco
          ansible-playbook playbook.yml -l **customer_abc**

.. hint::

    Ansible as shipped by many distribution is currently suffering from an
    incompatibility with our S3-compatible storage:

      Failed to get bucket tags: An error occurred (NoSuchTagSetError) when calling
      the GetBucketTagging operation: Unknown

    Should you see this error, it's easiest to patch Ansible locally to
    work around the issue. You have to find ``s3_bucket.py`` locally and
    patch it as shown `here <issue-150>`_. The file is likely somewhere
    in ``/usr``::

      find /usr -name s3_bucket.py

.. hint::

    When setting up the primary test system, "${CUSTOMER_NAME}test",
    be sure to run the playbook for the production system too. This
    because, once the test system is configured, Ansible will
    reconfigure the production system to reuse the Docker image
    used by the test system.


Update and Verify Installation Entry in BO
------------------------------------------

* update status
* set server


Add Customer Module
-------------------

:doc:`/framework/configuration/modules/add-customer-module`

(This is done last as one cannot start an installation localy without
running Ansible first. It creates the S3 bucket used locally too.)

.. rubric:: Footnotes

.. [#f1] By default, Ansible does check if *installation_name == customer_name* to
         decide if an installation is a production system and it will use that
         information to adjust the default settings. (See *installation_type*
         variable in *config.yml*.)


.. _common.yaml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
.. _Ansible Repository: https://git.tocco.ch/admin/repos/ansible
.. _issue-150: https://github.com/ansible-collections/amazon.aws/pull/150/files
