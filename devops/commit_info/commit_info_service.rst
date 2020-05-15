Commit-Info-Service
===================

The commit-info-service uses data from the following sources and combines them to generate useful output that is
accessible over its rest api.

- Tocco backoffice rest api to get a the list of installations and the installation urls
- Jira rest api to retrieve issues / information about issues
- The nice2 git repository (locally checked out) to get information about commits
- ``/status-tocco`` of a selected nice2 installation to get the currently installed revision of an installation
- ``/nice2/rest/modules`` of a selected nice2 installation to get a list of all modules

Repository
----------

The codebase of the commit-info-service tool is hosted in a private gitlab repository and may be found using the
following url. Please contact operations if you do not have access to it.

https://gitlab.com/toccoag/commit-info-service

CI/CD
-----

The gitlab ci/cd is used. The configuration can be found in the configuration file ``.gitlab-ci.yml`` in the repository.

A deployment is automatically started when a merge request is merged.

Hosting / Accessing Logs
------------------------

The commit-info-tool is run as in a docker container on our openshift platform. To access it you have to login and
switch to the "toco-commit-info-service".

The project can be managed as described in :ref:`OpenShiftBasics`. Please find below some commands that may be used to
manage "toco-commit-info-service".

    .. code-block:: bash

        #switch to project toco-commit-info-service
        oc project toco-commit-info-service

        #list pods
        oc get pods

        #read logs of a specific pod
        oc logs commit-info-35-jmmb6

        #edit deployment config for the commit-info pod
        oc edit dc/commit-info

        #restart commit-info
        oc scale dc/commit-info --replicas=0
        oc scale dc/commit-info --replicas=1


Debugging tips
--------------

- Installations are cached in the tool, if an installation is added or an installation path was amended a restart may be required to see imediate results
- Check if the url on the backoffice installation is correct, reachable and the ``status-tocco`` and ``nice2/rest/modules`` endpoints are reachable
- Check if the git repository is properly updated (check logs)
- Check if API Keys to backoffice and JIRA are valid (``TOCCO_API_USERNAME``, ``TOCCO_API_PASSWORD``, ``JIRA_API_USERNAME``, ``JIRA_API_TOKEN`` in ``dc/commit-info``)

REST-API
--------

Authentication
^^^^^^^^^^^^^^
There is a statically configured username + password that is used to authenticate when connecting to the commit-info
REST-API. Username and password may be found in the following deployment config environment variables:

    .. code-block:: text

        REST_API_USERNAME
        REST_API_PASSWORD

Installation-Resource
^^^^^^^^^^^^^^^^^^^^^

Installation-Endpoint
*********************

    .. code-block:: text

        GET https://commit-info-service.tocco.ch/installation
        GET https://commit-info-service.tocco.ch/installation/{instance}

Retrieves a specific or a list of all active installations from backoffice and returns them. Returns ``instance``
(technical name), ``label`` and ``url`` of the selected installations. To save time and resources, these installations
are cached for 6 hours as they usually do not change.

Commit-Endpoint
***************

    .. code-block:: text

        GET https://commit-info-service.tocco.ch/installation/{instance}/commit/{hash}

Checks if a given commit ``{hash}`` is installed on a given installation ``{instance}``

1. retrieves the installation url from the installation endpoint
2. get the installed revision from the status-tocco page using the retrieved url
3. uses git to determine whether the installed rev contains the given commit: ``git merge-base --is-ancestor {commit-hash} {instance-rev}``

Returns ``commitHash``, ``commitMessage``, ``installation`` and a boolean ``isInstalled``

Delta-Endpoint
**************

    .. code-block:: text

        GET https://commit-info-service.tocco.ch/installation/{instance1}/delta?rev={rev}
        GET https://commit-info-service.tocco.ch/installation/{instance1}/delta?installation={instance2}

Lists all commits that are on one instance and are not yet on the other installation or rev (api automatically
determines which one is newer). Only lists commits that are relevant for the given installation.

1. retrieves the installation urls of ``{instance1}`` and ``{instance2}`` (if ``installation=`` was used) from the installation endpoint
2. get the installed revisions from the status-tocco page using the retrieved url
3. use git to check which revision is older: ``git merge-base --is-ancestor {instance1-rev} {delta-rev}``
4. use git to get all commits between the two: ``git log --abbrev-commit --name-only --full-index --no-merges --date=iso-strict {older-rev}..{newer-rev}"``
5. get installed modules of {instance1}
6. remove irrelevant commits based on changed files. Only keep commits that changed files of core-modules, installed optional-modules or the selected customer module
7. retrieves information to all connected jira issues from the jira rest api

Returns ``instance1-rev``, ``delta-rev``, ``older-rev``, ``newer-rev`` and a list of commits (``commitId``, ``author``,
``commitTimestamp``, ``comitMessage``, ``changedFiles``) containing information about the related issues (``key``,
``summary``, ``status``, ``projectType``)

Issue-Resource
^^^^^^^^^^^^^^

Search-Endpoint
***************

    .. code-block:: text

        POST https://commit-info-service.tocco.ch/issue/search
        {
            "searchTerm" : "{searchTerm}"
        }

This may be used to search for issues from the jira rest api. Returns ``key``, ``summary``, ``status``, ``projectType``
for each issue found.

Issue-Endpoint
**************

    .. code-block:: text

        GET https://commit-info-service.tocco.ch/issue/{key}

Get issue details for a given issue-key from the jira rest api. Returns ``key``, ``summary``, ``status``,
``projectType``.

Commit-Endpoint
***************

    .. code-block:: text

        GET https://commit-info-service.tocco.ch/issue/{key}/commit

Get all commits that contain an issue key in the commit message: ``git log --all --abbrev-commit --name-only --full-index --no-merges --date=iso-strict --grep={key}``

Returns ``commitId``, ``author``, ``commitTimestamp``, ``comitMessage``, ``changedFiles`` for each commit.