Server Access (SSH/KVM/noVNC)
=============================

SSH Access
----------

========================  ==============================  ================================================  =======================================
 Hosts                     Login                           Example                                           Management
========================  ==============================  ================================================  =======================================
 \*.tocco.cust.vshn.net    | ${FIRST_NAME}.${LAST_NAME}    | ``ssh peter.gerber@db1.tocco.cust.vshn.net``    :ref:`Puppet <vshn-ssh-access>`
 \*.tocco.ch               | tocco (non-priviledged)       | ``ssh tocco@app01.tocco.ch``                    `Ansible <_ssh-server-access-ansible>`
                           | tadm (root via sudo)          | ``ssh tadm@app01.tocco.ch``
========================  ==============================  ================================================  =======================================

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



Allow SSH Access using Ansible
``````````````````````````````

See :ref:`ssh-server-access-ansible`.


Allow SSH Access using Puppet
``````````````````````````````

See :ref:`vshn-ssh-access`.


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
