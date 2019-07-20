S3 Tools
==========


s3cp
----
Description: copy between bucket-disc, bucket-bucket, disc-bucket

Functions:

1. copy all data from one bucket to another
2. copy all data from a bucket to a disc
3. copy data on a disk to a bucket

Command: s3cp

 .. parsed-literal::
    Argument: choices: ``bucket2bucket`` | ``bucket2disk`` | ``disk2bucket``
    Argument: ``source-bucket`` example: tocco-nice-master
    Argument: ``target-bucket`` example: tocco-nice-test222


s3rm
----

Description: delete everything in a s3 bucket

Command: s3rm

 .. parsed-literal::

    Argument: ``s3-bucket`` example: tocco-nice-test222
    Argument: ``--force``


n2s3fsck
--------

Description: Handling for correct alignment between the nice-overlay and the bucket.

Command: n2s3fsck


 .. parsed-literal::

    Argument: ``--no-password``, don't ask for a password
    Argument( ``--show-missing-on-db``
    Argument( ``bucket``, main bucket
    Argument( ``--overlay``, overlay bucket
    Argument( ``database``database to process
    Argument( ``--user``,database user
    Argument( ``--host``,database host
    Argument( ``--port``, database port
    Argument( ``--s3_cfg``, S3cmd config file to use'
    Argument( ``--force``
    Argument( ``--delete-unused-objects-from-bucket``, delete all objects from the bucket which are not used in the database
    Argument( ``--copy-from-overlay-2-main-bucket``, copy missing objects from the overlay bucket to the main bucket


