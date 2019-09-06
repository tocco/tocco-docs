Migration
=========

.. _history_migration:

2.19: History + S3
------------------

The History migration is mandatory in 2.19.
The S3 migration is optional and can be made in versions 2.19 or later.
The empty history databases and S3 buckets for all existing customers have already been :ref:`created <create_new_customer>`.

You can only do the S3 migration together with the history migration or after the history migration is done.

For bigger customers the migration takes multiple days. Because of this you have to do a pre migration before the actual migration. The pre migration can be started at any time and doesn't impact the running installation.

#. The migration script is located on a local vm::

    ssh tocco@migration

#. Open a new `screen <https://wiki.ubuntuusers.de/Screen>`_ session for each migration::

    screen -S ${INSTALLATION}

#. Start the migration.

   Show all options::

    migration --help

   You have to specify if you want to migrate the history, S3 or both.
   This is done by setting the optional arguments :code:`--history`, :code:`--s3`.

   You also have to specify the location of the customer. This either :code:`vshn` or :code:`nine`

   Starting the script without the :code:`--finalize` flag will do the pre migration.
   **Only use the --finalize flag during the actual migration of the whole installation**

   Example::

    migration --history --s3 vshn ${DB_NAME}

#. Check the progress of the running migrations::

    docker ps
    docker logs {CONTAINER_NAME}
