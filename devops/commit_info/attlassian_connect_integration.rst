Atlassian-Connect-Integration
=============================

The atlassian-connect-integration repository contains the "jira plugin" that is shown as the "Git Commits" tab in
https://toccoag.atlassian.net/

Repository
----------

The codebase of the jira plugin is hosted in a private gitlab repository and may be found using the following url.
Please contact operations if you do not have access to it.

https://gitlab.com/toccoag/atlassian-connect-integration

CI/CD
-----

The gitlab ci/cd is used. The configuration can be found in the configuration file ``.gitlab-ci.yml`` in the repository.

A deployment is automatically started when a merge request is merged.

Functionality
-------------

The ``atlassian-connect-integration`` project is an ``Atlassian Connect Add-On`` based on Spring Boot.
It uses the  `atlassian-connect-spring-boot-starter <https://bitbucket.org/atlassian/atlassian-connect-spring-boot/src/master/>`_ library
provided by Atlassian.

The library provides (among other things) the following features:

    * it serves the plugin descriptor (``atlassian-connect.json``) which is the entry point of the application
    * authentication of JIRA users using JWT

For more information about developing JIRA add-ons check out the `documentation <https://developer.atlassian.com/cloud/jira/platform/integrating-with-jira-cloud/>`_.
See also the `getting started guide <https://developer.atlassian.com/cloud/jira/platform/getting-started/>`_ that explains
how to locally develop an add-on using a JIRA development instance. We currently have one `JIRA development instance <https://tocco-dev.atlassian.net/>`_,
contact the development team for access (the number of users is limited as it is a free development instance).

Plugin descriptor
^^^^^^^^^^^^^^^^^

The plugin descriptor (``atlassian-connect.json``) is the entry point of the add-on. A URL to this file needs to be provided
when the add-on is installed.
The relevant part of the descriptor is the following:

.. code::

    "modules": {
        "jiraIssueTabPanels": [
          {
            "url": "/jira/commit-view/{issue.key}",
            "weight": 100,
            "name": {
              "value": "Git Commits"
            },
            "key": "git-commit-tab"
          }
        ]
      }

This defines that we want to add a new tab to the issue view and the content of that tab should be
loaded from ``/jira/commit-view/{issue.key}`` (``{issue.key}`` will be replaced by the JIRA issue key
that is currently open).

REST Endpoints
^^^^^^^^^^^^^^

This add-on provides several REST endpoints:

    * The main entry point is the CommitListViewController (``GET /jira/commit-view/{key}``) which loads
      the view ``commit-list.html``. This loads a React JS App that contains the tab content and makes
      additional REST calls to the plugin.
    * InstallationController and IssueController return the actual data about installations and commits.
      These endpoints do not really contain any logic and simply forward the call to the ``commit-info-service``
      tool.

Authentication
^^^^^^^^^^^^^^

Authentication is provided by the Atlassian library.
By default all REST endpoints are secured and can only be accessed using a valid JWT token (which is provided
by JIRA).
If authentication should be disabled for a certain endpoint (for example for development/test purposes)
the ``IgnoreJwt`` annotation can be used.

Deployment
----------

The app is deployed in the ``toco-commit-info-service`` OpenShift project.
After the add-on has been deployed it needs to be added to the Atlassian Marketplace using
a `vendor account <https://marketplace.atlassian.com/manage/vendors/1217087/addons>`_ (contact the development
team for access).

The process of how to publish the add-on to the marketplace und how to install the private add-on
into to the productive JIRA instance is described `here <https://developer.atlassian.com/platform/marketplace/installing-cloud-apps/>`_.

