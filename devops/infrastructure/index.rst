Infrastructure Overview
=======================

Overview of our infrastructure operated by VSHN.

Appuio Platform
---------------

Platform based on OpenShift which in turn is built around Kubernetes.

Tocco
^^^^^

For every customer we operate an independent instance and every
instance is in an independent OpenShift project with the name
toco-nice-${INSTALLATION_NAME}.

On a Kubernetes-level our setup looks something like this:

.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Provided Service
     - Management
   * - dc/nice
     - There are two containers:

       ========== =====================================
        Name       Description
       ========== =====================================
        nice       | The Tocco Application
                   |
                   | Our main application
        nginx      | Nginx reverse proxy
                   | Provides compression, caching and
                   | support for custom headers.
       ========== =====================================
     - **Ansible** except for:

       * Memory requests and limits
       * | Some java properties (configured via env. vars)
         | are set manually
       * PVCs (see below)

   * - | *svc/nice*
       | *route/nice*
       | *route/nice-\**
     - | There is one service, *svc/nice*, that
       | handles all traffic going to our application.

       | There is always a route called *route/nice* using
       | ${INSTALLATION_NAME}.tocco.ch as FQDN. Additional
       | routes may exist that follow the naming convention
       | *nice-${FQDN}*.

       | All routes use ACME to issue and renew TLS
       | certificates. All connections are upgraded to
       | HTTPS via ``insecureEdgeTerminationPolicy: Redirect``
       | and `HSTS`_ header.
     - **Ansible**

       | A ``tocco.ansible-managed: "true"`` annotation
       | is used to ensure Ansible does not touch routes
       | created manually or by other tools (like the
       | ACME controler).
       |
       | No such manually create routes exist as of
       | today.
   * - | Docker registry
       | *is/nice*
     - | Docker image of our main application, Tocco.
       | Built and then pushed from outside OpenShift
       | by our CD tool `TeamCity`_.

       | Pushed images are deployed automatically
       | using an *imageChange* trigger.
     - **Ansible**
   * - is/nginx
     - There are two global nginx images in use:

       =============== ==============================
        Name / Image    Description
       =============== ==============================
        nginx:stable    Production Nginx
        nginx:latest    Staging Nginx
       =============== ==============================

       | Both images reside in the project
       | ``toco-shared-imagestreams``.
     - | **Manually**
       |
       | Updating and promoting from staging to
       | production is done manually.
   * - monitoring
     - | Currently only a simple http check is used to check
       | if our status page (``/status-tocco``) returns code
       | 200 within a given time.
     - | **Ansible**
       |
       | Ansible generates a definition in the Puppet
       | Hiera format as required by VSHN's monitoring.
       | The configration is then committed to
       | :hierra-repo:`monitoring.yaml`.
   * - logging
     - | Logs are written to stdout as JSON. Those logs
       | are then collected and made available using
       | Elastic Search and Kibana.
     -
   * - DNS
     - | Domains managed by us are hosted at Nine. However,
       | many domains are hosted by customers themselves
       | or third parties in the customer's name.
     - Manually via web interface.
   * - PVC for :abbr:`LMS (learning management system)`
     - Our e-learning solution  stores files in a PVCs.

       | There are currently only 6 system which have
       | this in use in use.
     - Manually
   * - PVC for out-of-memory dumps
     - | For debugging purposes, we use PVCs to extract
       | memory dumps from Tocco.
     - Manually

.. _HSTS: https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security
.. _TeamCity: https://www.jetbrains.com/teamcity/

Tocco Manual
^^^^^^^^^^^^

Manual of Tocco consisting of static HTML and hosted on Appuio.

.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Provided Service
     - Management
   * - dc/documentation-${VERSION}
     - | For every version of Tocco, a manual
       | is released and deployed separately.
     - Manually via template
   * - route/documentation-${VERSION}
     -
     - Manually via template
   * - monitoring
     -
     - **Puppet**

       Added to VSHN's Puppet config manually.
   * - logs
     - Default Nginx logs written to stdout
     -
   * - DNS
     -
     - Manually


Jira Commit Info Service
^^^^^^^^^^^^^^^^^^^^^^^^

Integration of deployment, merge and commit information into Jira. See also
:doc:`/devops/commit_info/commit_info_service`.

.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Provided Service
     - Management
   * - | dc/commit-info
       | jira-addon
     -
     - Manually
   * - pvc/repository
     - | Clone of our main Git repository. Used
       | to display commit and deployment information
       | in Jira.
     - Manually
   * - | route/\*
       | svc/\*
     -
     - Manually
   * - is/\*
     -
     - Deployed via GitLab CI


Sonar
-----

`SonarQube <https://sonarqube.org>`_ code inspection tool.

An instance of sonarQube is running to analyze the source code
of Tocco.


.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Provided Service
     - Management
   * - dc/\*
     -
     - Manually
   * - is/\*
     -
     - Deployed manually

Dashboard
---------

Simple `dashboard`_ for our developers.

.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Provided Service
     - Management
   * - dc/\*
     -
     - Manually
   * - is/\*
     -
     - Deployed via Travis

.. _dashboard: https://github.com/tocco/tocco-dashboard

Address Provider
----------------

External `addressprovider`_ service

The service is deployed via GitLab CI and the service definition is managed
via Ansible (:ansible-repo-file:`playbook <services/playbook.yml>`,
:ansible-repo-dir:`role <services/roles/address-provider>`).

Deployment::

    $ cd ${ANSIBLE_REPO/services
    $ ansible-playbook playbook.yml -t address-provider

.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Provided Service
     - Management
   * - dc/\*
     -
     - Manually
   * - | route/\*
       | svc/\*
     -
     - Manually
   * - is/\*
     -
     - | Production: Deployed via TeamCity
       | Test: Deployed via GitLab

.. _addressprovider: https://gitlab.com/toccoag/address-provider


.. _image_service:

Image service
-------------

We use a service called `imaginary <https://github.com/h2non/imaginary>`_ running in its own pod. The Openshift project
containing the service is called ``toco-image-service``. All calls to the service require a header ``API-Key`` be used,
containing the key as defined in ``image_service_api_key`` in :term:`secrets2.yml`.

From the backend we call the ``/crop`` endpoint of the service to generate thumbnails. Other endpoints may be used freely
if the need ever arises, nothing is blocked.

The service is deployed via GitLab CI and the service definition is managed
via Ansible (:ansible-repo-file:`playbook <services/playbook.yml>`,
:ansible-repo-dir:`role <services/roles/image-service>`).

Deployment::

    $ cd ${ANSIBLE_REPO/services
    $ ansible-playbook playbook.yml -t image-service

.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Provided Service
     - Management
   * - dc/\*
     -
     - Manually
   * - is/\*
     -
     - Deployed manually

Managed Servers - VSHN
----------------------

Postgres
^^^^^^^^

`Postgres`_ database server used for the primary database
of Tocco.

.. list-table::
   :header-rows: 0
   :widths: 10 30

   * - Version
     - Postgres 12
   * - Required extensions
     - `lo`_, `pg_trgm`_, `uuid-ossp`_

       | Extensions are installed on database
       | via Ansible (`CREATE EXTENSION`_).
   * - Backups
     - 7 daily database dumps + 4 weekly
   * - Users / databases
     - Databases and users are managed by Ansible.

.. _Postgres: https://postgresql.org
.. _lo: https://www.postgresql.org/docs/current/lo.html
.. _pg_trgm: https://www.postgresql.org/docs/current/pgtrgm.html
.. _uuid-ossp: https://www.postgresql.org/docs/current/uuid-ossp.html
.. _CREATE EXTENSION: https://www.postgresql.org/docs/current/sql-createextension.html


Solr
^^^^

`Apache Solr`_ used to provide full-text search capabilities.

.. list-table::
   :header-rows: 0
   :widths: 10 30

   * - Version
     - Solr 7.4
   * - Authentication
     - | Via `Basic Authentication Plugin`_ providing
       | HTTP Auth support.
   * - Transport security
     - | HTTPS with TLS cert signed by globally trusted
       | authority.
   * - Backups
     - | 7 daily + 4 weekly
       |
       | Implemented using LVM snapshots.
   * - Cores (AKA indexes)
     - Created via Ansible


.. _Apache Solr: https://lucene.apache.org/solr/
.. _Basic Authentication Plugin: https://lucene.apache.org/solr/guide/8_4/basic-authentication-plugin.html


Mail Relay
^^^^^^^^^^

SMTP server used for outgoing mails.

The mail server admits all incoming mails. Restricting
Sender domains/addresses is left up to Tocco.

.. list-table::
   :header-rows: 0
   :widths: 10 30

   * - Transport Security
     - | STARTTLS with TLS cert signed by globally
       | trusted authority.
   * - `DKIM`_
     - | Mails are signed using DKIM. Generally, one
       | and the same key is used for all mails.
       | However, for a few domains we use another
       | key to avoid name clashes.
       |
       | See also :doc:`/devops/mail/dns_entries`

.. _DKIM: https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail
