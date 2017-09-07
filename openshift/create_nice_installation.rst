Create a Nice Installation
==========================

A template is used to create a new installation. It can be found in the `Ansible Git Repository`_.

.. _Ansible Git Repository: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=openshift/nice-template.yml


Step by Step Instructions
-------------------------

#. Create a new project

    .. figure:: create_nice_installation/create_project.png
        :scale: 60%

    Go to the `Create Project`_ page of VSHN and create a new project [#f1]_.

    .. hint::

        ``toco-`` is prefixed automatically. Hence, the project name is going to be ``toco-nice-${INSTALLATION}``.

.. _Create Project: https://control.vshn.net/openshift/projects/appuio%20public/_create

#. Clone the `Ansible Git Repository`_

   .. code::

       git clone ssh://USER@git.tocco.ch:29418/ansible.git

#. Go to the ``openshift`` directory within the repository.

   .. code::

       cd ansible/openshift

#. Switch to the newly created project

    .. code::

        oc project toco-nice-${INSTALLATION}

#. Allow pulling images from project toco-shared-imagestreams [#f2]_

    .. code::

        oc policy add-role-to-user system:image-puller system:serviceaccount:toco-nice-${INSTALLATION}:default --namespace=toco-shared-imagestreams

#. Create all resources required

   .. code::

       oc process -f nice-template.yml -v CUSTOMER=${CUSTOMER} -v … | oc create -f -

   Parameter are specified using ``-v KEY=VALUE``, this is the list of **mandatory** parameters:

   =================== ===============================================================================================
    Key                 Value
   =================== ===============================================================================================
    CUSTOMER            Name of the customer (e.g. agogis or ecap but never :strike:`agogistest` or
                        :strike:`ecaptest`).

    INSTALLATION        Name of the installation (e.g. ecap or ecaptest)

                        :subscript:`The name of a test system MUST end in "test"!`

    RUN_ENV             Run environment which must be one of ``production`` or ``test``.
   =================== ===============================================================================================

   Additionally, these optional parameters are available. **(Default values should suffice mostly.)**

   ===================== ==========================================================================================
    Key                   Value
   ===================== ==========================================================================================
    FLUENTD_TARGET        URL to the :term:`Fluentd` logging service.

                          :subscript:`Leave blank to log to /app/var/log/nice.log instead.`

    JAVA_MEM              Max. memory available to Java (e.g. ``1.5g`` or ``512m``).

    DB_PASSWORD           Password for database access.

                          :subscript:`Randomly generated if left off.`

    DB_SERVER             URL to the Postgres database server.

    DOCKER_REGISTRY_URL   URL to the Docker image registry.

    HSTS_SECS             ``max-time`` used for Strict-Transport-Security HTTP header.

    NGINX_IMAGE_NAME      Name of the :term:`Nginx` Docker image.

    SMTP_RELAY            Hostname of SMTP relay.

    SOLR_DISK_SPACE       Persistent disk space available to :term:`Solr` (e.g. ``512m`` or ``5g``).

    SOLR_IMAGE_URL        Name of the Solr Docker image.
   ===================== ==========================================================================================

#. Issue an SSL Certificate

    Issue a SSL certificate for ${CUSTOMER}.tocco.ch which is created by the template. See :ref:`issue-ssl-certificate`
    for instructions.

#. Add additional Routes / Hostnames if Needed

    See :ref:`add-route`

.. important::

    The installation needs also to be :ref:`created in Teamcity <create-installation-in-teamcity>`.

.. note::

  The installation is automatically started once :term:`CD` pushes an image to the Docker registry.


.. rubric:: Footnotes

.. [#f1] An unlimited number of project is included in dedicated APPUiO.

.. [#f2] Nginx and Solr images, which are used by all Nice projects, are in toco-shared-imagestreams.

