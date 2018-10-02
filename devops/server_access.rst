Server Access (SSH/KVM/noVNC)
=============================

SSH Access
----------

========================  ==============================  ================================================  =============
 Hosts                     Login                           Example                                           Management
========================  ==============================  ================================================  =============
 \*.tocco.cust.vshn.net    | ${FIRST_NAME}.${LAST_NAME}    | ``ssh peter.gerber@db1.tocco.cust.vshn.net``    `Puppet`_
 \*.tocco.ch               | tocco (non-priviledged)       | ``ssh tocco@app01.tocco.ch``                    `Ansible`_
                           | tadm (root via sudo)          | ``ssh tocco@app01.tocco.ch``
========================  ==============================  ================================================  =============

.. hint::

    Some servers are not directly reachable from the outside. Use git or backup02 as jump host in that case::

        ssh -J tocco-proxy@git.tocco.ch ${TARGET_HOST}

        # or

        ssh -J tocco-proxy@backup02.tocco.ch:32711 ${TARGET_HOST}

.. hint::

    You can set the default user name used by ssh in ``~/.ssh/config``.

    Example configuration::

       Host *.tocco.cust.vshn.net
           User peter.gerber

       Host *.tocco.ch
           User tocco


.. _Ansible:

Allow SSH Access using Ansible
``````````````````````````````

Access can can be granted via `roles/ssh-key-sync/files/ssh_keys`_ in the Ansible repository.

Changes can be deployed via Ansible::

    cd ${ANSIBLE_GIT_ROOT}
    ansible-playbook -i inventory playbook.yml

.. hint::

    Users with role ``@user`` have access as user tocco. User with role ``@root`` as tadm and tocco.

.. _roles/ssh-key-sync/files/ssh_keys: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=roles/ssh-key-sync/files/ssh_keys


.. _Puppet:

Allow SSH Access using Puppet
``````````````````````````````

Puppet configuration can be found the `tocco_hieradata repository`_. Access is defined in the ``users`` section within
the different config files (e.g. in ``database.yml`` for database servers and ``infrastructure/solr.yml`` for Solr
servers).

Users are managed via this files with exception of those in ``database.yml``. For those add the user in Ansible (see
previous section) and generate the content for the YAML file::

    cd ${ANSIBLE_REPOSITORY}
    playbooks/ssh_key_sync/ssh-pubkey-parser hiera ${HIERADATA_REPOSITORY}/playbooks/ssh_key_sync/ssh_keys ~/src/vshn/tocco_hieradata/database.yaml

.. hint::

    Users that are part of the group ``toccoroot`` can use sudo to obtain root priviledges.

.. _tocco_hieradata repository: https://git.vshn.net/tocco/tocco_hieradata/tree/master


KVM
---

For server operated by us, KVMs can be accessed at https\://${HOST}kvm.tocco.ch. For instance, the KVM
for app03.tocco.ch is located at https://app03kvm.tocco.ch.

.. warning::

    KVMs are only accessible from the Tocco office network.


Managing Virtual Machines (Proxmox)
-----------------------------------


Proxmox is used for our in-office infrastructure. The VMs can be managed, including terminal access, via webinterface. See
:doc:`internal_servers/servers_and_services/provided_services/index` for a list of available servers

.. hint::

    You can acces the webinterface at https\://${HOST}. Available host are listed the document linked above.

.. warning::

    Proxmox web interfaces are only accessible from the Tocco office network.
