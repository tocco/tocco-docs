.. _large_objects:

Large objects
=============

The :nice:`BinaryAccessProvider <ch/tocco/nice2/persist/spi/binary/BinaryAccessProvider>` provides access to
:nice:`Binary <ch/tocco/nice2/persist/entity/Binary>` instances.

While the actual data may be saved to different data stores, the metadata is saved in the ``_nice_binary`` (and partially
``nice_resource``) table in Postgres.

Currently there are two different implementations:

    * The :nice:`DbBinaryAccessProvider <ch/tocco/nice2/persist/backend/jdbc/impl/DbBinaryAccessProvider>` is the default
      and stores the data in a large object of Postgres.
    * The :nice:`S3AccessProvider <ch/tocco/nice2/optional/s3/storage/S3AccessProvider>` is provided by the ``optional/s3storage``
      module and can access any S3 compatible data store. See also :doc:`../s3/s3`.

The main task of a :nice:`BinaryAccessProvider <ch/tocco/nice2/persist/spi/binary/BinaryAccessProvider>` is to read,
write and delete :nice:`Binary <ch/tocco/nice2/persist/entity/Binary>` instances.

The :nice:`AbstractBinaryAccessProvider <ch/tocco/nice2/persist/spi/binary/AbstractBinaryAccessProvider>` is the base class
that handles all modifications of the ``_nice_binary`` table, while sub-classes save the actual binary data into their data store.

Loading
-------

The method ``loadBinary(HashCode, Connection)`` is called from the :nice:`BinaryUserType <ch/tocco/nice2/persist/hibernate/usertype/BinaryUserType>`
and loads the binary in two steps:

    * The metadata is loaded from the Postgres tables (that's why we need a :java:`Connection <java/sql/Connection>`)
    * The actual binary data is lazily loaded when ``openStream()`` is called

.. warning::

    The :java:`Connection <java/sql/Connection>` passed to this method belongs to the Hibernate transaction and
    should never be committed manually!

There is a second method ``loadBinary(HashCode, long, String, String)`` that can be used to create a :nice:`Binary <ch/tocco/nice2/persist/entity/Binary>` instance
without accessing the Postgres database, if the metadata is already known.

Saving
------

``storeBinary(Binary, Connection)`` saves a :nice:`Binary <ch/tocco/nice2/persist/entity/Binary>` instance to the data store.
Because the binary is referenced by its hash code, the same file is only saved once, even if it's referenced multiple times
by the database.

Therefore at first it is checked if this file already exists. If not, an entry in the ``_nice_binary`` table is created
and then the data itself is saved to the data store.

All changes made through the passed :java:`Connection <java/sql/Connection>` are transactional and might be rolled back.

Deleting
--------

``removeBinary(HashCode, Connection)`` tries to remove the entry in the ``_nice_binary`` table.
Since we only save one copy of the same file to the data store, a row in ``_nice_binary`` might be referenced multiple times.
In order to know when the row can be safely deleted, a ``reference_count`` column is maintained by a trigger (see ``binary_reference_trigger.sql``).

Objects Stored in DB
^^^^^^^^^^^^^^^^^^^^

When the ``reference_count`` is zero, the binary will automatically be deleted by the :nice:`DeleteUnreferencedBinariesBatchJob <ch/tocco/nice2/dms/impl/maintenance/DeleteUnreferencedBinariesBatchJob>`.
The large object itself will be removed by the built-in `lo_manage`_ trigger.

Objects Stored in S3
^^^^^^^^^^^^^^^^^^^^

The :nice:`S3AccessProvider <ch/tocco/nice2/optional/s3/storage/S3AccessProvider>` is largely based on the functionality
above, but there are some differences:

    * Because S3 is independent of the JDBC transaction, there might be orphaned objects in the data store if the JDBC
      transaction is rolled back, after a new object has been stored.
    * When a binary is removed (by the :nice:`DeleteUnreferencedBinariesBatchJob <ch/tocco/nice2/dms/impl/maintenance/DeleteUnreferencedBinariesBatchJob>`)
      it is only marked as deleted (column ``removed_at``) and removed later by an external tool.
    * S3 offers the possibility to create a `pre-signed URL`_ to an object that is valid for a certain amount of time (see ``Binary.Store#getUrl()``),
      this allows downloading the object directly from the S3 server instead of causing unnecessary traffic for the
      nice installation.

BinaryHashingService
--------------------

The :nice:`BinaryHashingService <ch/tocco/nice2/persist/binary/BinaryHashingService>` abstracts the conversion of a
binary into its hash code. This allows different :nice:`BinaryAccessProvider <ch/tocco/nice2/persist/spi/binary/BinaryAccessProvider>`
to use different hashing strategies.

    * ``hashFunction()`` defines the hash function to be used
    * ``getStringGenerator()`` can be used to encode the hash (for example with BASE64)

BinaryDataAccessor
------------------

The :nice:`BinaryDataAccessor <ch/tocco/nice2/persist/hibernate/binary/BinaryDataAccessor>` is a service to efficiently
query the ``_nice_binary`` and ``nice_resource`` tables.

This service is necessary, because currently the ``_nice_binary`` table is not mapped by Hibernate, which means it cannot
be referenced by the query builder.

It is used by the query builder, so that binary metadata can be queried efficiently without causing a query for every single
binary.


.. _lo_manage: https://www.postgresql.org/docs/9.5/lo.html
.. _pre-signed URL: https://docs.aws.amazon.com/AmazonS3/latest/dev/ShareObjectPreSignedURL.html
