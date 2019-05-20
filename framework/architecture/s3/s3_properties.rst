.. _s3_architecture:

Architecture
============


Diagram Showing the S3 Setup used in a Dev Environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An overlay bucket is provided for the dev. environment.
This prevents developers from writing data to the production bucket during development.


The S3 data storage is created for Tocco with the following buckets.

1. A bucket for the installation
2. A bucket for the developers, which is overwritten by configuration.



.. image:: ./resources/s3_architecture.jpg
    :width: 600pt


S3 Integration details
^^^^^^^^^^^^^^^^^^^^^^

In the Tocco framework, ``createClient()`` is called from
src/nice2/optional/s3storage/impl/src/main/ java/ch/tocco/nice2/optional/s3/storage/S3AccessProvider.java   and overrides the bucket for the overlay.


To get access to the overlay bucket, a valid credentials file must be located locally in the folder ``~/.aws``

.. note::

    The credentials that are specified in the shared credentials (``s3.properties``) file include
    Priority over credentials in the AWS CLI configuration file (``~/.aws``).


View the S3 Architecture :ref:`s3_properties`