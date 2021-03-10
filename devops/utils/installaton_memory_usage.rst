###################
Memory Usage as CSV
###################

This document describes an Ansible playbook that generates a CSV file
containing detailed information about the memory reserved for an
application.


About the Generated CSV
=======================

The information is provided as CSV and contains these columns:

===================== =====================================================
 Column                Description
===================== =====================================================
 Installation          | Installation name.
 Memory (MiB)          | This is the amount of memory requested on
                       | Kubernetes. This should be about ~80% of the
                       | memory that's actually being used. The rest is
                       | spare to be able to deal with spikes in usage.
===================== =====================================================

Generating the CSV
==================

.. hint::

    This documentation assumes you've setup Ansible already. See
    :ref:`setup-ansible`.

Generate CSV::

    cd ${ANSIBLE_REPO}/tocco
    ansible-playbook playbooks/memory_usage.yml -e output=data.csv -f 50

This stores the CSV file at *playbooks/data.csv*.
