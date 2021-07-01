CI Pipeline
===========

Workflow cherry-pick
--------------------

**Naming**

- source branch: branch on which the CI job is running (e.g. ``nice-releases/227``)
- target branch: branch to which the commits should be cherry-picked (next release branch or master; e.g. ``nice-releases/228``)
- auto merge tag: each release branch has a git tag like ``auto-merge/${VERSION}``. All commits before this tag (including the commit itself) are already cherry-picked (if necessary)

**Workflow**

- setup variables (auto merge tag, source branch, nice version)
- configure git and clone repo (only source & target branch and all tags of these branches)
- get commit id which belongs to the auto merge tag of the source branch (e.g. tag ``auto-merge/227``)
- checkout source branch
- retrieve commits which should be cherry-picked. Such a commit has a ``Cherry-pick:Up`` in the commit message body and are after the auto merge tag in the history (=not already cherry-picked)
- stop if nothing should be cherry-picked
- checkout target branch
- cherry pick commit by commit
- release packages (see `Workflow Releasing`_ where all packages are released on the target branch if the changelog is not empty)
- create and push new branch ``releasing/auto-xxxxxxxx``
- create github PR
- move auto merge tag to head of source branch

Workflow Releasing
------------------

*In the CI pipeline the script is executed with the ``--auto`` argument*

- setup variables (last release tag, last version, changelog, nice version, release tag)
- define next version of package (increase hotfix version)
- check if git tree is clean (else stop)
- check if changelog of package is not empty (else stop)
- commit changelog
- run ``yarn publish`` which creates a new chore commit
- move npm dist tag to published hotfix version

Fix failing cherry-pick job
---------------------------

For example there could be an auto merge conflict. In such a case there is log message like this:

.. code-block:: bash

    CONFLICT (content): Merge conflict in packages/entity-browser/src/routes/list/containers/ListViewContainer.js

**Workflow for fixing** (in the example the source branch is ``nice-releases/227`` and the target branch ``nice-releases/228``) **:**

- check log which commits are cherry-picked (see line 2 und 3):

.. code-block:: bash

    c2b6b34cdca570fd36bdef75682b702a9702938b is commit id of tag auto-merge/227
    00f9bed1a0969efceaaed43e10c952f1e0fd57f0 has 'Cherry-pick: Up' in commit message
    952f1e0fd57f000f9bed1a0969efceaaed43e10c has 'Cherry-pick: Up' in commit message
    Switched to branch 'nice-releases/228'

- checkout the target branch
- cherry pick the commits with ``git cherry-pick ${COMMIT_ID}`` and fix conflicts
- Create a PR
- if the PR is approved release all necessary packages (and create a new PR for the release commits)
- Move the auto merge tag to the head of the source branch

.. code-block:: bash

    git checkout ${HEAD_COMMIT_ID}
    git tag -f auto-merge/${SOURCE_BRANCH_NICE_VERSION}
    git push origin auto-merge/${SOURCE_BRANCH_NICE_VERSION} -f
