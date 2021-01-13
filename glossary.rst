:orphan:

Glossary
========

.. glossary::

    Ansible Repository
        Git repository describing the configuration of our servers in a format
        that Ansible understands.

        It can be found at https://git.tocco.ch/gitweb?p=ansible.git;a=summary.

    Ansible Vault
        Used to store passwords and other secrets securely.

        Two are currently in use, :term:`secrets.yml` and :term:`secrets2.yml`.

        See section *Ansible Vault* in `this document <https://git.tocco.ch/gitweb?p=ansible.git
        ;a=blob;f=docs/ansible/basics.rst>`_ for a detailed documentation.

    BO
    Backoffice
        This is what we call our own installation of Nice2 that can be reached at https://www.tocco.ch/tocco.

    application property
        Application properties can be used to configure Nice. They are simple key, value pairs and look like this:

        .. code::

            my.property=configuration value

        The chapter :ref:`app-properties-in-openshift` describes how to set parameters in OpenShift. Alternatively, a
        property can be changed for a customer by adjusting the ``application.properties`` file. See next paragraph.

        While developing, you can edit the application.properties in the Nice Git repository located at
        ``customer/${CUSTOMER}/etc/application.properties``. If you don't want to commit the changes you can create
        an ``application.local.properties`` in the same directory. Settings made the the *local* properties file
        override the settings in the regular properties file.

    FTL
    Freemarker
    Freemarker Template Language
        Templating language used throughout Nice2. In particular, report and mail content is mostly generated
        using this language.

    HiveApp
        Extension of :term:`HiveMind`.

    HiveMind
        HiveMind is a service and configuration microkernel used within Nice2. The `official web page
        <https://hivemind.apache.org/hivemind1/index.html>` has some more details.

        Within Nice2 HiveMind is used to configure listeners, batch jobs, default property value, reports
        and much more.

    JFrog's Artifactory
    Artifactory
        Tocco hosts its own JFrog's Artifactory, a Maven software repository. During build, all artifacts are fetched
        from there. Either, the package is uploaded to the artifact directly or a :term:`Remote Repository` can be
        configured.

        Our Artifactory can be found at https://mvn.tocco.ch.

    BURP

        Backup archiving tool using on some of our servers. See https://burp.grke.org for details.

    continuous delivery
    CD
        Continuous delivery is used to deploy our Nice installations.

        Our CD is powered by TeamCity and can be found at https://tc.tocco.ch.

    container
        A :term:`docker image` running in a :term:`pod`.

        Configuration is part of the :term:`deployment config`.

    DB refactoring
        In the context of Nice, the DB migration in generally referred to as DB refactoring.

    deployment config
    DC
        The deployment config describes the containers associated with it. This includes image sources, resource limits,
        open ports, roll out strategy, triggers, etc.

         Accessible via ``oc {get|describe|edit|…} dc …``.

    docker image
        An image that contains an application and all run-time dependencies except the OS.

    Employee Short Name
        Usually the first two letters of an employee's first and last name combined. For
        instance, *Jane Doe* becomes *jado*.

        These are the initials used in Slack and can also be found in :term:`BO` on *Person*
        as *Kurzbez.*

    exposed port
        Port that is made available to other pods or services.

        This is configured in the :term:`deployment config`.

    hibernate collection
        A collection that is persisted to the database (a one-to-many or many-to-many association)

    image stream
    IS
        Describes a docker repository. Pushing a docker image to it can be used to trigger an automatic deployment.

        Accessible via ``oc {get|describe|edit|…} is …``.

    image stream tag
        Describes a docker image tag. Defaults to ``latest``.

        Accessible via ``oc {get|describe|edit|…} imagestreamtag …``.

    JasperReports
        `JasperReports`_ is a report technology still used by some legacy reports. The reports
        use the \*.jrxml file extension.

        New reports should use :term:`wkhtmltopdf`.

    Maven Archive
        An archive (\*.tar.gz) of the whole application including all dependencies and other resources
        needed to run the applicaiton.

        Such an archive can be created using this command::

            mvn -pl customer/${CUSTOMER} -am install -T1C -DskipTests -P assembly

        See also `Apache Maven Assembly Plugin <https://maven.apache.org/plugins/maven-assembly-plugin/>`_.

    Nginx
       `Nginx`_ is the web server used for as reverse proxy in front of Nice.

        Nginx is running in the same :term:`pod` as Nice.

        .. _Nginx: https://nginx.org/en/

    persistent volume claim
    PVC
        A persistent volume that can be mounted into one or more containers.

        Accessible via ``oc {get|describe|edit|…} pvc …``.

    pod
    PO
        A pod is one instance of the containers described in its :term:`deployment config`.

        Accessible via ``oc {get|describe|edit|…} pod …``.

    pre-hook pod
        A pre-hook pod is a :term:`pod` that is executed during rollout, before executing the actual pod. In our setup,
        it is used for :term:`DB refactoring` and some startup checks. For more details, see
        `Pod-based Lifecycle Hook`_ in the OpenShift documentation.

        .. _Pod-based Lifecycle Hook: https://docs.okd.io/latest/dev_guide/deployments/deployment_strategies.html#pod-based-lifecycle-hook

    Operations Public channel
        Slack channel `operations_public <https://app.slack.com/client/T0S4PA46T/C2R6SKHGC>`_ that can be used to contact
        the operations team.

    PD4ML
        `PD4ML`_ is a Java-based HTML to PDF converter used by some legacy reports.

        New reports should use :term:`wkhtmltopdf`.

    Remote Repository
        In :term:`Artifactory`, Remote Repositories can be configured. For such repositories, Artifactory will forward
        requests to the configured upstream repository and cache the result for later use.

        Remote Repositories can be configured in **Admin** → **Remote**.

    Replication Controller
    RC
        The replication controller is responsible to ensure the specified number of replicas is running at all times.

        There is one RC per deployment. Use ``oc describe rc …`` to see the configuration (:term:`DC`) that was used for a deployment.

        Accessible via ``oc {get|describe|edit|…} pod …``

    secrets.yml

       Used to store passwords, API keys and other secrets. Encrypted using
       :term:`Ansible Vault` and stored in the :term:`Ansible Repository`.

       **secrets.yml** contains secrets required for setting up servers and services
       other than Nice. See also :term:`secrets2.yml`.

       View secrets::

           $ cd ${ANSIBLE_REPO}
           $ ansible-vault view secrets.yml

       Edit secrets::

           $ cd ${ANSIBLE_REPO}
           $ ansible-vault edit secrets.yml

    secrets2.yml

       Used to store passwords, API keys and other secrets. Encrypted using
       :term:`Ansible Vault` and stored in the :term:`Ansible Repository`.

       **secrets2.yml** contains secrets required for setting up Nice and related
       services. As general rule, secrets required so setup an installation go
       here. See also :term:`secrets.yml`.

       View secrets::

           $ cd ${ANSIBLE_REPO}/tocco
           $ ansible-vault view secrets2.yml

       Edit secrets::

           $ cd ${ANSIBLE_REPO}/tocco
           $ ansible-vault edit  secrets2.yml

    service
    SVC
        Used to make a service available in the network. It provides a DNS name for a service in a way that hides the
        fact that the service may be provided by several pods (multiple replicas).

        Accessible via ``oc {get|describe|edit|…} svc …``.

    Solr
        Solr is a search engine, Nice uses it to provide full-text search.

        Every Nice installation runs exactly one Solr :term:`pod`.

    Solr core
        Indexes in :term:`Solr` are known as cores.

    route
        Provides a route to a service. This is used to make a service reachable via internet.

        Accessible via ``oc {get|describe|edit|…} route …``.

    wkhtmltopdf
        A command line tool for converting HTML into PDF. Within Nice it is used to generate PDF reports.

        See :doc:`/framework/architecture/reports/wkhtmltopdf` and :doc:`/framework/configuration/reports`.


.. _JasperReports: https://community.jaspersoft.com/project/jasperreports-library
.. _PD4ML: https://pd4ml.com
