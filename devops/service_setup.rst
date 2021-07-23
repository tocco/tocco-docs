.. highlight:: bash

Set up Application/Service on OpenShift
#######################################

Walkthrough for setting up a service on OpenShift. In this
guide the service is first set up manually and then exported
to Ansible to ease management and recreation of the service.

Prepare a Docker Image
======================

First, you need a Docker image that can be deployed.

Sometimes upstream images from Docker Hub, or elsewhere, can be
used without modification.

However, if an upstream image isn't an option, check out `Get
Started`_ in the official Docker documentation, in particular
if you're new to Docker. Many languages and build automation
tools have their own set of standards for building a Docker
image; if you're looking for a more advanced guide, you may
be better off looking for a specific guide. `Spring Boot
with Docker`_ would be an example for such a guide.

When building your own image, ensure all necessary runtime
configuration options are made available via environment
variables.

.. note::

    OpenShift differs from running bare Docker in some
    regards. You should be aware of the following caveats:

    **Missing Write Permission**

    OpenShift runs containers as a non-root user by default.
    Some images available may not work as non-root and it
    may be necessary to adjust permissions on certain files
    and directories if the application needs write access
    to any of them. Grant write access by assigning the
    file or directory to group *0* and granting that group
    write access. To this end, add the following to
    `Dockerfile`_::

        RUN chgrp -R 0 /some/directory \
          && chmod -R g+w /some/directory

    **Username Required**

    By default, the user as which a container is run, does not
    have a username or a $HOME directory. However, some application
    require it. To do so, grant write access to */etc/passwd* to
    group *0* in `Dockerfile`_::

        RUN chgrp 0 /etc/passwd \
          && chmod g+w /etc/passwd

    Then, as part of the entrypoint script, add an entry
    for the user::

        USERNAME=user
        HOME=/APP
        echo "${USERNAME}:x:$(id -u):0::${HOME}:/sbin/nologin" >> /etc/passwd

    For Java applications, an alternative approach is to set the *user.home*
    property. Setting it to */* is often sufficient.


    **See Also**

    `OpenShift Container Platform-Specific Guidelines`_.


Create Project
==============

Usually, it's best to have an OpenShift/Kubernetes project
for any independent service. Services belonging together may
also be placed in the same project.

Here is how to create a project:

#. Go to `APPUiO Projects`_
#. Select *APPUiO Public*
#. *Create New*
#. Select *system:serviceaccount:toco-serviceaccounts:ansible* as admin
#. Give the project a name (a *toco-* prefix is added automatically)

   Use hyphens to separate words (e.g. *image-service*).
#. Accept terms
#. *Create*
#. Find project in list and edit it

   Page may have to be reloaded for project to appear.
#. Edit *Admin(s)*
#. Enable group *tocco* (on/off switch)
#. *Save*

.. tip::

    If a test system is desired, the test system should have the
    same name as production except that it has an additional *-test*
    suffix.

    It's recommended that you only create production manually and
    use Ansible to create the second project, the test system. This
    way you can be sure the project can be (re-)created via Ansible.


Initial, Manual Deployment
==========================

.. note::

    **Technical note:**

    The Docker Image may be fetched as a result of scaling, pod
    evacuation, pod restart or compute node restart. Consequently,
    the availability of the registry is highly important. To
    minimize the risk depending on an external registry has,
    all images are stored on OpenShift's registry.

    Also note that Docker Hub, Docker's default registry, has
    a strict rate limit which we'd likely hit when a compute
    node is evacuated, fails or is restarted.

.. note::

    **${PROJECT_NAME}**

        Name of the project you created or are reusing. Note
        that Openshift and Kubernetes use the terms project and
        namespace interchangeably.

    **${SERVICE_NAME}**

        A name given to the service within the project. If you
        created a project for just this service, reuse
        ${PROJECT_NAME} as ${SERVICE_NAME} excluding the
        *toco-* prefix. That is, when ${PROJECT_NAME} is
        toco-my-project, ${SERVICE_NAME} should be
        *my-project*.

#. Login to Docker Registry::

       # if not logged in yet
       oc login

       docker login -u any --password-stdin registry.appuio.ch < <(oc whoami -t)

#. Either a) fetch and upload an existing image (e.g from Docker Hub)::

       docker pull ${IMAGE_FROM_ELSEWHERE}
       docker tag ${IMAGE_FROM_ELSEWHERE} registry.appuio.ch/${PROJECT_NAME}/${SERVICE_NAME}
       docker push registry.appuio.ch/${PROJECT_NAME}/${SERVICE_NAME}

   or b) build an image locally and upload it::

        cd ${DIRECTORY_WITH_DOCKERFILE}
        docker build -t registry.appuio.ch/${PROJECT_NAME}/${SERVICE_NAME} .
        docker push registry.appuio.ch/${PROJECT_NAME}/${SERVICE_NAME}

#. Switch to the project::

        oc project ${PROJECT_NAME}

#. Create application/service on OpenShift::

       oc new-app --image-stream ${SERVICE_NAME}

   See also `Creating an application from an image`_.

#. Configure service via environment variables::

       oc set env deployment ${SERVICE_NAME} KEY1=VALUE1 KEY2=VALUE2

   Settings for publicly available images are usually documented in
   the README of their respective repository.

   You can also list the environment variables::

       oc set env deployment ${SERVICE_NAME} --list

#. Add DNS record for service:

   See :ref:`dns-managed-by-us`. Add record for the test system too.

#. Expose service to the public::

       oc expose service abc --hostname=${SERVICE_NAME}.tocco.ch

   .. hint::

        It's also possible to make a service available at a specific path.
        For instance, a service can be made available at
        *https://service.tocco.ch/api/v2*::

            oc expose service ${SERVICE_NAME} --hostname ${SERVICE_NAME}.tocco.ch --path api/v2

#. Issue a TLS certificate::

       oc annotate route/${SERVICE_NAME} kubernetes.io/tls-acme=true

   This can take some time. See :ref:`acme-troubleshooting` if certificate
   isn't issued within 15 minutes.

   .. tip::

        By default, the service is only available via HTTPS and any request
        via HTTP will return an error. This is recommended for any service
        where the user is not expected to access the service directly (by
        typing the address in a browser's address bar). It will help detecting
        when HTTP is used by accident.

        If and **only if** required, HTTP request may be redirected to HTTPS::

            oc patch route abc -p '{"spec": {"tls": {"insecureEdgeTerminationPolicy": "Redirect"}}}'

#. Tell clients to always use HTTPS::

       oc annotate route/${SERVICE_NAME}} haproxy.router.openshift.io/hsts_header=max-age=62208000

   See `Strict-Transport-Security`_ and `Enabling HTTP strict transport security`_

#. Add persistent storage (if required)

   All storage is non-persistent by default. If you need any file or
   directory to survive an application restart, create a persistent
   volume::

       oc set volume deployment/${SERVICE_NAME} --add --name ${VOLUME_NAME} --claim-name ${VOLUME_NAME} --claim-size ${N}G --mount-path ${PATH}

   .. tip::

      Some images out there require that temporary files can be written to a
      certain directory but the user doesn't have write access to it. Remember
      that the user in the container isn't root on OpenShift (unlike bare Docker).
      In this case, you can add an empty, writable directory::

          oc set volume deployment/abc --add --name ${VOLUME_NAME} --type=EmptyDir --mount-path ${PATH}

      Unlike persistent storage, this storage won't survive a restart and it's
      content isn't shared among instances.

#. Request CPU and Memory

   CPU and memory should always be requested. Set it to the expected CPU and memory
   usage expected during normal operation::

       oc set resources deployment ${SERVICE_NAME} --requests cpu=N,memory=${N}Mi

   For instance, to request 0.05 CPUs and 256 MiB of memory pass this::

       --requests cpu=0.05,memory=256Mi

   This information is required by Kubernetes to assign resources properly. See
   `Motivation for CPU requests and limits`_.

   .. hint::

       **--limit**

       Use ``--limit cpu=${N},memory=${N}Mi`` to limit maximum resource consumption.
       This shouldn't usually be required.

       Be warned that any application exceeding the memory limit is terminated
       immediately.

Test The Service
================

At this point the service should be operational. Verify everything works
as excepted.

Should the service not be available, try to debug the issue:

* Use https\:// explicitly.

  Service is not available via http\:// by default.

* Check project status::

      oc status

* Check logs of any running pod (if any)::

      oc logs deployment/${SERVICE_NAME}

  Or check log of a specific pod::

      oc get pods
      oc logs ${POD_NAME}

* On permission errors, see warnings in `Prepare a Docker Image`_

* List all resources::

      oc get resources

  Use column *NAME* as ${RESOURCE} in the following commands.

* Check resources (pay attention to *Events* at the bottom)::

      oc describe ${RESOURCE}

* Edit resource::

      oc edit ${RESOURCE}


.. highlight:: yaml

Export Service to Ansible
=========================

Services are located in :ansible-repo-dir:`services` directory within the Ansible
repository. Services are structured like this:

================================= ========================================================
 File / Directory                  Description
================================= ========================================================
 services/roles/${SERVICE_NAME}/   | Ansible role for managing the service
                                   |
                                   | See `Role directory structure`_ in
                                   | the Ansible documentation.

 services/playbook.yml             | Playbook for all services
                                   |
                                   | Call role from here for test and production.
                                   |
                                   | Use tag to only configure a specific
                                   | service:

                                   .. code::

                                       cd ${ANSIBLE_REPO}/services
                                       ansible-playbook playbook.yml -t ${SERVICE_NAME}
================================= ========================================================

#. Add entry (for prod and test) in :ansible-repo-file:`playbooks.yml
   <services/playbook.yml>`.

   It should looks something like this (for production)::

      - name: ${HUMAN_READBLE_SERVICE_NAME} production
        import_role:
          name: ${SERVICE_NAME}
          vars_from: prod
        tags: [ ${SERVICE_NAME}, ${SERVICE_NAME}-prod ]

#. Create *services/${SERVICE_NAME}/vars/{prod,test}.yml*.

   This files contain variable specific to production or
   test. They should contain, at least, the following::

       # Name of the OpenShift/Kubernetes project
       k8s_project: ${PROJECT_NAME}

       # Hostname at which service is reachable
       #
       # {{ … }} is a Jinja2 template and will be replaced
       # by Ansible at runtime.
       hostname: '{{ k8s_project }}.tocco.ch'

   .. tip::

        Variables shared by production and test should be added
        to *services/${SERVICE_NAME}/defaults/main.yml*. Variables
        defined in the *vars/* directory, like the ones above,
        will override the ones in *defaults/*.

#. Create resources in *services/${SERVICE_NAME}/tasks/main.yml*

   See :ansible-repo-file:`services/roles/image-service/tasks/main.yml`
   for an example.

   **Manually add the following**

   a) Create project::

       - name: create project
         vshn_openshift:
           token: '{{ secrets2.vshn_openshift_token }}'
           project: '{{ k8s_project }}'
           admin_uids: [ 'system:serviceaccount:toco-serviceaccounts:ansible' ]
           admin_gids: [ 'Cust tocco' ]

   b) Create account for use by GitLab (used to push Docker image)::

       # Creating a service account without secrets. Those are created
       # automatically if omitted.
       #
       # See https://docs.openshift.com/container-platform/3.6/dev_guide/service_accounts.html#dev-managing-service-accounts
       - name: create service account for use by GitLab
         k8s:
           host: '{{ k8s_endpoint }}'
           api_key: '{{ secrets2.openshift_ansible_token }}'
           resource_definition: "{{ lookup('template', 'service_account_gitlab.yml') }}"

       - name: create rolebinding
         k8s:
           host: '{{ k8s_endpoint }}'
           api_key: '{{ secrets2.openshift_ansible_token }}'
           resource_definition: "{{ lookup('template', 'rolebinding_gitlab.yml') }}"

   c) Create *rolebinding_gitlab.yml* and *service_account_gitlab.yml* in
      *services/${SERVICE_NAME}/templates/*. Copy
      :ansible-repo-file:`rolebinding_gitlab.yml
      <services/roles/image-service/templates/rolebinding_gitlab.yml>`
      and :ansible-repo-file:`service_account_gitlab.yml
      <services/roles/image-service/templates/service_account_gitlab.yml>`
      from the image-service, verbatim.

   d) Export the remaining resources.

      At least these resources ${RESOURCE} need to be
      exported:

      * deployment/${SERVICE_NAME}
      * route/${SERVICE_NAME}
      * service/${SERVICE_NAME}

      Other resources, you created, may need to be exported too.
      I.e. additional routes, services, secrets, etc. If you're
      unsure, have a look at the list of all resources:

      .. code-block:: bash

           oc get all

      Generated resources like *Pods* and *replicasets* need not
      to be exported.

      For every resources, add a task to tasks/main.yml. Example
      for a route::

         - name: create route
           k8s:
             host: '{{ k8s_endpoint }}'
             api_key: '{{ secrets2.openshift_ansible_token }}'
             namespace: '{{ k8s_project }}'
             resource_definition: "{{ lookup('template', 'route.yml') }}"

      See :ansible-repo-file:`services/roles/image-service/tasks/main.yml`
      for more examples.

      Next export the resource (e.g. route, service, deployment)::

          oc get ${RESOURCE} -o yaml


      .. todo::

           Use ``--export``, which strips the output, once that option
           available on our platform.

      Place the resulting definition in *services/${SERVICE_NAME}/templates/*.
      Use the same file name as you used in *tasks/main.yml* (``route.yml``
      in the example above).

      Postprocess the resulting YAML files:

      * Strip unwanted metadata like the *uid*, *selfLink* or
        *resourceVersion*.
      * Replace project name with the this Jinja2 template
        ``{{ k8s_project }}``.
      * Replace any values that need to be different in prod and
        test with ``{{ … }}`` templates and ensure the variables
        are defined in *vars/* (see above).
      * Replace any secret (like password) with ``{{ secrets2.${SOME_NAME} }}``
        and add a secret with name ${SOME_NAME} to :term:`secrets2.yml`.

      See :ansible-repo-dir:`services/roles/image-service/templates/`
      for examples.

.. highlight:: bash

#. Run Ansible for production::

       cd ${ANSIBLE_REPO}/services
       ansible-playbook playbook.yml -t ${SERVICE_NAME}-prod

#. Run Ansible for test::

       cd ${ANSIBLE_REPO}/services
       ansible-playbook playbook.yml -t ${SERVICE_NAME}-test

   Ensure the test system is available. (Issuing a TLS certificate
   may take some time.) See also `Test the Service`_.

   Remember to switch to the right project for debugging::

       oc project ${PROJECT_NAME}-test


Setup Repository for Deploying Docker Image
===========================================

#. Create a repository on `GitLab`_ for the application you're deploying
   if there is none yet.

#. Set up build pipeline on GitLab

   Either, using an upstream image:

       If you're using an upstream image (e.g. from Docker Hub), setup
       an new repository with a pipeline for deploying production and
       test. Use `.gitlab-ci.yml of image-service`_ as a template.

   or, alternatively, when building your own image:

       If you're building your own image, be sure anything needed
       to build it is in a repository. Any Dockerfile, resources and
       possibly the application/service itself. Then use the `.gitlab-ci.yml
       of address-provider`_ as a template for a pipeline to deploy
       production and test.

#. Create environment variables on GitLab containing the tokens needed
   to push the Docker images. In the example *.gitlab-ci.yml* linked above,
   those are called OC_TOKEN_PROD and OC_TOKEN_TEST for production and
   test respectively.

   #. Obtain the token

      Find the token name (first item listed in *Tokens*):

      .. parsed-literal::

          $ oc describe serviceaccount gitlab
          Name:                gitlab
          Namespace:           toco-image-service
          Labels:              <none>
          Annotations:         <none>
          Image pull secrets:  gitlab-dockercfg-w8g9l
          Mountable secrets:   gitlab-token-l7krz
                               gitlab-dockercfg-w8g9l
          Tokens:              **gitlab-token-l7krz**
                               gitlab-token-wlk5f
          Events:              <none>

      Get secret:

      .. parsed-literal::

           $ oc describe secret **gitlab-token-l7krz**
           Name:         gitlab-token-l7krz
           Namespace:    toco-image-service
           Labels:       <none>
           Annotations:  kubernetes.io/service-account.name: gitlab
                         kubernetes.io/service-account.uid: 53686829-bf86-11eb-888f-fa163e3ec73a
           
           Type:  kubernetes.io/service-account-token
           
           Data
           ====
           ca.crt:          2137 bytes
           namespace:       18 bytes
           service-ca.crt:  3253 bytes
           tokens:          **eyJhbGciOi<yes this really long string is the token you want>nL4JSZmHCg**

   #. On GitLab, go to the repository and find Settings → CI/CD → Variables → Add Variable

      Create a variable called OC_TOKEN_PROD and OC_TOKEN_TEST with the respective tokens.
      Be sure to check *Protect variable* during creation.


Documentation
=============

* List service on :doc:`/devops/infrastructure/index`.

  * Concisely describe for what service is used.
  * Mentioned how to deploy it.
  * Mention where to find the definition in Ansible.

  A more detailed, full-page documentation may be appropriate for some services. Add a link
  to that document here. Also, link any relevant upstream documentation.

* Add a link to the documentation on Read the Docs in GitLab's README file.


Updates
=======

Services need to be updated. Even if the application isn't changed, dependencies and the
underlying Docker images should be updated.

Define a schedule for updating and make sure the people responsible for doing so are
informed. The Address-provider and image-service are both updated as part of creating
a release branch. Consider this as an option and if appropriate update the documentation
at :doc:`/devops/continuous_delivery/new_release_branch` with instructions.


.. _Get Started: https://docs.docker.com/get-started/
.. _Spring Boot with Docker: https://spring.io/guides/gs/spring-boot-docker/
.. _.gitlab-ci.yml of address-provider: https://gitlab.com/toccoag/address-provider/-/blob/master/.gitlab-ci.yml
.. _.gitlab-ci.yml of image-service: https://gitlab.com/toccoag/image-service/-/blob/master/.gitlab-ci.yml
.. _Dockerfile: https://docs.docker.com/engine/reference/builder/
.. _OpenShift Container Platform-Specific Guidelines: https://docs.openshift.com/container-platform/3.10/creating_images/guidelines.html#openshift-container-platform-specific-guidelines
.. _Creating an application from an image: https://docs.openshift.com/container-platform/4.7/applications/application_life_cycle_management/creating-applications-using-cli.html#applications-create-using-cli-image_creating-applications-using-cli
.. _APPUiO Projects: https://control.vshn.net/appuio/projects
.. _Role directory structure: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-directory-structure
.. _Motivation for CPU requests and limits: https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/#motivation-for-cpu-requests-and-limits
.. _Strict-Transport-Security: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
.. _Enabling HTTP strict transport security: https://docs.openshift.com/container-platform/4.6/networking/routes/route-configuration.html#nw-enabling-hsts_route-configuration
.. _GitLab: https://gitlab.com/toccoag
