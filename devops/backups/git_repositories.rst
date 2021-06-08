Git Repository Backups
======================

GitLab Mirrors
--------------

Many of our repositories on Gerrit have mirrors on GitLab. The mirrors are kept
in sync via :ansible-repo-file:`Gerrit hook <roles/gerrit/files/hook>`.

Mirrored repositories:

=========================== ======================================================
 Repository                  Mirror
=========================== ======================================================
 ansible                     https://gitlab.com/toccoag/ansible
 nice2                       https://gitlab.com/toccoag/nice2
 nice2-documentation         https://gitlab.com/toccoag/nice2-documentation
 nice2-documentation-build   https://gitlab.com/toccoag/nice2-documentation-build
 tocco-ha                    https://gitlab.com/toccoag/tocco-ha
=========================== ======================================================

.. tip::

    You can aadd an additional remote to an existing repository like this::

        # Add remote named "gitlab"
        $ git remote add gitlab git@gitlab.com:toccoag/nice2.git
        $ git fetch gitlab

        # switch remote of existing local branch
        $ git checkout releases/3.0
        $ git branch -u gitlab/releases/3.0

.. hint::

   You need to be member of the group *toccoag* to be able to see and access
   the mirrors.
