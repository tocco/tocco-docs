.. _s3_architecture:

Architecture
============


S3 Integration Diagram only for Developer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An overlay bucket is provided for the developers.
This prevents developers from writing data to the productive bucket during testing.


The S3 datastorage is created for Tocco with the following buckets.

1. A bucket for the client
2. A bucket for the developers, which is overwritten due to configuration.



.. image:: ./resources/s3_architecture.jpg
    :width: 600pt


S3 Integration details
^^^^^^^^^^^^^^^^^^^^^^

In the Tocco framework, ``createClient()`` is called from
src/nice2/optional/s3storage/impl/src/main/ java/ch/tocco/nice2/optional/s3/storage/S3AccessProvider.java  
overrides the bucket for the overlay.


To get access to the overlay bucket, a valid credential-file must be located locally in the folder ``~/.aws``

.. note::

    The credentials that are specified in the shared credentials file include
    Priority over credentials in the AWS CLI configuration file.


View the S3 Architecture :ref:`s3_properties`