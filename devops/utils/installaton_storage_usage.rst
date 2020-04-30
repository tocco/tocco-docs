####################
Storage Usage as CSV
####################

This document describes an Ansible playbook that generates a CSV file
containing detailed information about the storage used by a
customer/installation.


About the Generated CSV
=======================

The information is provided as CSV and contains these columns:

===================== =====================================================
 Column                Description
===================== =====================================================
 Customer              | Customer to which the installation belongs.

 Installation          | Installation name.

 DB Size               | Size of the database as reported by Postgres.
                       | This corresponds to the actual storage used
                       | on disk and may include dead rows, indexes
                       | other internally used data.

 History DB Size       | Size of the history DB as reported by Postgres.

 S3 Size               | Size of the S3 bucket as reported by Cloudscale.
                       | All installations of a customer share a bucket.
                       | Thus, this size is only listed on one, the first,
                       | installation of a customer..

 Solr Size             | Size of the Solr search index.

 Total                 | Space used by an installation. S3 storage is
                       | included for the first installation of a customer
                       | only.

 Total Customer        | Total storage used by a customer.
===================== =====================================================

All number are provided in GiB, rounded down.


Generating the CSV
==================

.. hint::

    This documentation assumes you've setup Ansible already. See
    :ref:`setup-ansible`.

Generate CSV::

    cd ${ANSIBLE_REPO}/tocco
    ansible-playbook playbooks/storage_usage.yml -e output=data.csv -f 50

This stores the data in *playbooks/data.csv*.

You can generate a report for a limited number of customers using ``-l`` like
this: ``-l customer-agogis,customer-bk``.

.. hint::

    ``-f 50`` is a precaution to limit the number of simultaneous requests. 50
    is likely fine but in case you hit a ratelimit, lower it.
