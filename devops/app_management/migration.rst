Migration
=========

.. _history_migration:

2.19: History + S3
------------------

The History migration is mandatory in 2.19.
The S3 migration is optional and can be made in version 2.19 or later.
The empty history databases and S3 buckets for all customer are created
automatically via Ansible.

You can only do the S3 migration together with the history migration or
after the history migration is done.

For bigger customers, the history migration takes multiple days and the S3
migration several hours. Because of this, you have to do a pre-migration
before the actual, final migration.  The pre-migration can be started at
any time and doesn't impact the running installation.


Pre-Migration / Incremental Migration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

   Example::

       migration --history --s3 vshn ${DB_NAME}

#. Check the progress of the running migrations::

       docker ps
       docker logs {CONTAINER_NAME}


Preparations for Final Migration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. In order to enable S3, add this to ``customer/${CUSTOMER}/pom.xml``::

    <dependency>
      <groupId>ch.tocco.nice2.optional.s3storage</groupId>
      <artifactId>nice2-optional-s3storage-module</artifactId>
      <version>${project.parent.version}</version>
      <type>appmodule</type>
      <scope>compile</scope>
    </dependency>

#. Confiure S3 in ``customer/${CUSTOMER}/etc/s3.properties``::

    s3.main.endpoint=https://objects.cloudscale.ch
    s3.main.bucketName=tocco-nice-${CUSTOMER}

These changes must be merged before running CD during the final migration.


Final Migration
^^^^^^^^^^^^^^^

#. Create a database dump

   **Dump now and not during the deployment. --finalize will apply irreversible changes.**

#. During final migration run the same command again but this time with an
   additional ``--finalize``::

       migration --history --s3 --finalize ${DB_NAME}

   .. hint::

        Once run with *--finalize*, it won't be possible to upload new documents
        and history won't be written any longer. Be sure to do this as short
        before the deployment as possible.

#. Update installation via CD to enable S3 and/or history DB

   Disable DB dump here.

   .. hint::

      If locking issues occur during the DB migration, stop the installation
      temporarily and kill all connections to the DB and then retry (``oc
      rollout retry dc/nice``).


Post-Migration
^^^^^^^^^^^^^^

This steps can be done at any time after the migration.

#. Remove objects from DB::

       ssh ${DB_SERVER} sudo -u postgres vacuumlo ${DB_NAME}

#. VACUUM selected tables

   .. code-block:: sql

       VACUUM pg_largeobject;
       VACUUM pg_largeobject_metadata;
       VACUUM _nice_binary;

   (Doing a regular VACUUM first to speed up VACUUM FULL which we want to be
   as fast as possible; it holds an exclusive lock.)

#. VACUUM FULL selected tables

   .. code-block:: sql

       VACUUM FULL pg_largeobject;
       VACUUM FULL pg_largeobject_metadata;

       SET statement_timeout TO '3 min';
       VACUUM FULL _nice_binary;  -- This may fail after three minutes with a timeout. If
                                  -- so, ignore it. We don't want to spend more time as
                                  -- this locks the table exclusively making all objects
                                  -- unavailable while it is running.

#. Check size of tables (in psql)::

       \dS+

   The tables *pg_largeobject* and *pg_largeobject_metadata* should now have a
   size of 0. If this isn't the case, run all the VACUUM commands again later.
   This can happen when there is still an active transaction for which some
   objects are still alive (=the transaction started before an object was
   removed by *vacuumlo*).


.. _recaptcha:

2.25: ReCaptcha
---------------

We now use `ReCaptcha v3 <https://www.google.com/recaptcha/intro/v3.html>`_ for a few use cases.
For each installation a client key and a secret (server) key needs to be configured.
Each key is tied to a set of URLs.

The keys for development (``localhost`` only) are configured in hivemodule.xml and won't work when deployed.
There is a default set of keys for the ``tocco.ch`` domain (incl. subdomains), which are automatically configured
through ansible.

See the 2.25 Migration comments in Backoffice for the links to the ReCaptcha keys (where also additional URLs can be added).

If the installation is running on a custom domain (anything other than \*.tocco.ch) all additional domains
need to be added to the ReCaptcha config. Currently this is only necessary when the login or password
update dialog are used in the custom domain.

If different keys need to be used for a certain installation the following properties need to be overridden:

The client key needs to be configured with the ``nice2.userbase.captcha.client.key`` property.
The secret key needs to be configured with the ``nice2.userbase.captcha.secret`` property.

These properties **must** be set via Ansible. The properties can set via the ``application_properties``
variable as described in :ref:`ansible-app-properties`. Either override them for a customer or single
installation, see :ref:`ansible-variable-precedence`.
