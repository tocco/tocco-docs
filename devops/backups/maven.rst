Maven / JFrog Artifactory Backups
=================================

.. todo::

   Mention and link still-to-write VM backups document.

Long-Term Backups on GitLab
---------------------------

Export / Backups:

There are automatically created, long-term backups of mvn.tocco.ch in the `maven-repo-backup repository`_ on Gitlab. The
backups have been exported using the `built-in import and export functionality`_.  See `Ansible Repository`_ for details.

Import:

#. Copy files to the server
#. Ensure user ``artifactory`` has read access (``chmod -R artifatcory $DIR``)
#. Login on https://mvn.tocco.ch
#. Go to Admin → Import & Export → Repositories
#. Import files from path

Import into ``~/.m2/repository``:

#. Copy files from ``$GIT_ROOT/repositories/*/*`` to ``~/.m2/repository``

.. _maven-repo-backup repository: https://gitlab.com/toccoag/maven-repo-backup
.. _Ansible Repository: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=roles/artifactory/tasks/backups.yml
.. _built-in import and export functionality: https://www.jfrog.com/confluence/display/RTF/Importing+and+Exporting#ImportingandExporting-RepositoriesImportandExport
