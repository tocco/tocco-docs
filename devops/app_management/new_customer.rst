Create new Customer
===================

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



Update and Verify Installation Entry in BO
------------------------------------------

* update status
* set server


Add Customer Module
-------------------

:doc:`/framework/configuration/modules/add-customer-module`

(This is done last as one cannot start an installation localy without
running Ansible first. It creates the S3 bucket used locally too.)


.. _common.yaml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/common.yaml
.. _Ansible Repository: https://git.tocco.ch/admin/repos/ansible
