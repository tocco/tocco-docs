Contribute
===========

Philosophy
-----------
Changes to the current version will be developed in separate feature branches. As soon as a feature is ready, a pull request is opened to rebase the feature branch into the master branch.
After a successful rebase the feature branch should be deleted. Only self-contained features should be rebased to master to keep master stable and packages always ready to be published.

How to fix bugs in older versions see `Maintaining older versions`_



Pullrequest
---------------
#. Create a remote branch that fits the following naming convention: ``pr/{DESCRIPTION-OF-CONTRIBUTION}``. 
#. Push commits to this branch. Set a commit message as described below.
#. Once all changes are pushed, create a pull request. The changes should never break a package and therefore must be self-contained.
#. The pull request will be verified by TravisCI and Codecov. If one of them returns a bad result, the problems have to be fixed.
#. Assign a reviewer manually.
#. Once the pull request is rebased, the branch must be deleted.


.. note::

  If a change is very large its recommended to create a feature branch and in the process of developing make small pullrequests (part) that are rebased in the main feature branch.

  .. code-block:: console

    git checkout -b pr/{DESCRIPTION-OF-CONTRIBUTION} pr/{DESCRIPTION-OF-PART-CONTRIBUTION}


Git Commit Msg
--------------
Similar to `Karma`_ commit messages follow this convention:

.. _Karma: http://karma-runner.github.io/0.10/dev/git-commit-msg.html

.. code-block:: console

  <type>(<scope>): <subject>

  <body>

  Changelog: <feature>
  Refs: <Jira Task Number>
  Cherry-pick: Up 

Message subject (first line)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First line must not be longer than 70 characters followed by a blank line. 

**<type> values**

========= =======================
feat      New feature
fix       Bug fix
docs      Changes to documentation or changelogs
style     Formatting etc.; no code change
refactor  Refactoring production code
test      Adding missing tests, refactoring tests; no production code change
chore     Updating webpack etc; no production code change
========= =======================



**<scope> values**

* If the changes affect a single package the scope is set to package name (e.g. login).
* If the change is a global or difficult to assign to a single package the parentheses are omitted.
* If changes affect the mono-repo itself, the scope is set to 'tocco-client'.

Message body
^^^^^^^^^^^^

* Should have a list structure
* Includes motivation for the change and contrasts with previous behavior
* Uses the imperative, present tense: “change” not “changed” nor “changes”


**Changelog**

If a release relevant feature is added with the commit, a Changelog line can be added.

.. code-block:: console 

  Changelog: Add feature XY

A commit can have several changelog entries but an entry needs to be on one single line.
The changelog entries are used to generate the changelog of the corresponding `<scope>` package.

**Refs**

At the moment there is no interface between Jira and the new client code. But with Refs a commit can be 
linked to a task which might be helpful to comprehend a change.

**Cherry-pick**

Should only be set in commits for older versions. See `Maintaining older versions`_ 


Maintaining older versions
---------------------------
Older Nice versions are using older releases of client packages. If a critical bug is found in an older package we need to fix it in that version. It's not possible to just
fix the bug in master an install the newest package since that could lead to compatibility problems if for example the rest endpoint changes. Furthermore we dont want to
deploy all new features with the bug fix.

For each Nice release there is a release branch in the tocco-client repository. These release branches are protected and require commits to be submitted via a pull request.

.. warning::

    Release branches have to be created parallel to the Nice releases and have to be used strictly!


Bug fixing
^^^^^^^^^^^
So if a bug is found, let's say in Nice version 2.17, we have to fix this bug in 2.17, 2.18, ... and master.
Assumed it's a critical bug, otherwise it will just be fixed in master with a pull request branch.

#. Find out the oldest yet supported version of Nice that contains the package with the bug.
#. Create a fix branch based on the release branch (e.g. ``git checkout -b pr/217/login/bug nice-releases/217``)
#. Commit fix to branch. Preferably with a regression test to verify the fix. (Add ``Cherry-pick: Up`` to the commit message body as ``Refs: TOCDEV-1`` (parsing is case insensitive and whitespaces are ignored but hyphen and semicolon are required) that the commit is automatically cherry picked and released in the versions 2.18 - master)
#. Create a pull request, wait until approved and rebase into release branch.
#. Checkout release branch and publish the affected package. It's important to not increment the PATCH version for hotfixes in older versions. Chances are that this version already exists on a newer branch. Therefore a --hotfix has to be added to the current version. For more info see `Naming`_ chapter and :ref:`releasing-script`.
#. Delete fix branch.
#. Merge release branch in next version and publish there as well. (see requirements for automation in step 3)
#. Repeat until hotfix is no more relevant or the bug is fixed in the newest version (master).

Naming
^^^^^^
============== ===========================================  ======
what            schema                                      example
============== ===========================================  ======
Release Branch nice-releases/niceversion                    nice-releases/217
Fix Branch     pr/niceversion/package/descr                 pr/217/login/image-bug
Hotfix Release currentversion-hotfixVersion.HotFixNumber    1.0.2-hotfix217.2
Release Tag    niceVersion                                  nice215
============== ===========================================  ======



Example
~~~~~~~

.. figure:: resources/release_branching.png

   Bug fix release Example (Created with draw.io, source xml in resource folder)

This examples shows two packages (Merge and Login) each with an individual release program.

Performed actions:

- Minor releases in master branch (feature branch are not show in diagram)
- Bug fix with fix branches in older version of Nice.
- Npm Tags (latest tags of master releases not shown).