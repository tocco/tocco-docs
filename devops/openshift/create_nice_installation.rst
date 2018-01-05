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
        Before you ask ``toco`` isn't a typo but rather the abbreviation that VSHN uses for Tocco.

.. _Create Project: https://control.vshn.net/openshift/projects/appuio%20public/_create

#. Assign project to group **tocco**

    .. figure:: create_nice_installation/tocco_group_1.png
        :scale: 60%

        show details for the project

    .. figure:: create_nice_installation/tocco_group_2.png
        :scale: 60%

        edit *admin*\(s)

    .. figure:: create_nice_installation/tocco_group_3.png
        :scale: 60%

        add **tocco** group to admins

#. Add service account (SA) to Project

    .. _add-sa-reference-label:

    .. figure:: create_nice_installation/vshn_control_sa_1.png
        :scale: 60%

        edit *editor*\(s)

    .. figure:: create_nice_installation/vshn_control_sa_2.png
        :scale: 60%

        add **system:serviceaccount:toco-serviceaccoutns:teamcity** SA to editors

    .. note::

        This Step is **mandatory** else the Installation can't be deployed via Teamcity-CI.
        If you'd forget this step, don't worry: It will print this error during Deployment:

        .. code::
           
           Error response from daemon: Get https://registry.appuio.ch/v2/: unauthorized: authentication required

        This error message doesn't always have to lead to the issue described.
        For further information about service accounts see :ref:`service-account-reference-label`.

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

       oc process -f nice-template.yml CUSTOMER=${CUSTOMER} … | oc create -f -

   Parameter are specified using ``KEY=VALUE``, this is the list of **mandatory** parameters:

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
    JAVA_MEM              Max. memory available to Java (e.g. ``1.5g`` or ``512m``).

    DB_PASS               Password for database access.

                          :subscript:`Randomly generated if left off.`

    DB_SERVER             URL to the Postgres database server.

    DOCKER_REGISTRY_URL   URL to the Docker image registry.

    HSTS_SECS             ``max-time`` used for Strict-Transport-Security HTTP header.

    SMTP_RELAY            Hostname of SMTP relay.

    SOLR_DISK_SPACE       Persistent disk space available to :term:`Solr` (e.g. ``512Mi`` or ``5Gi``).
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

