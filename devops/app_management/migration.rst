Migration
=========

.. _history_migration:

2.19: History + S3
------------------

The History migration is mandatory in 2.19.
The S3 migration is optional and can be made in versions 2.19 or later.
The empty history databases and S3 buckets for all existing customers have already been
:doc:`created </framework/configuration/modules/add-customer-module>`.

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


2.25: ReCaptcha
---------------

We now use `ReCaptcha v3 <https://www.google.com/recaptcha/intro/v3.html>`_ for a few use cases.
For each installation a client key and a secret (server) key needs to be configured.
Each key is tied to a set of URLs.

The keys for development (``localhost`` only) are configured as default and won't work when deployed.
There is a default set of keys for the ``tocco.ch`` domain (incl. subdomain).
See the 2.25 Migration comments in Backoffice for the links.

If the installation is running in a custom domain (anything other than *.tocco.ch) this additional domains
need to be added to the ReCaptcha config. Currently this is only necessary when the login or password
update dialog are used in the custom domain.

The client key needs to be configured with the ``nice2.userbase.captcha.client.key`` property.
The secret key needs to be configured with the ``nice2.userbase.captcha.secret`` property.
