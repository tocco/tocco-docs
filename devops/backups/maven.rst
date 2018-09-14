Maven / JFrog Artifactory Backups
=================================

.. todo::

   Mention and link still-to-write VM backups document.

Long-Term Backups on GitLab
---------------------------

There are manually created, long-term backups of mvn.tocco.ch in the `maven-repo-backup repository`_ on Gitlab. The
backups have been exported using the `built-in import and export functionality`_.

Export:

#. Login on https://mvn.tocco.ch
#. Create empty dir (``mkdir /tmp/export``)
#. Ensure it's writable by Artifactory (``chown artifactory /tmp/export``)
#. Go to Admin → Import & Export → Repositories
#. Check **Create .m2 Compatible Export**
#. Export to ``/tmp/export``
#. Copy ``/tmp/export/repositories/*`` to the maven-repo-backup repository

Import:

#. Copy files to the server
#. Ensure user ``artifactory`` has read access (``chmod -R artifatcory $DIR``)
#. Login on https://mvn.tocco.ch
#. Go to Admin → Import & Export → Repositories
#. Import files from path

Import into ``~/.m2/repository``:

#. Copy files from ``$GIT_ROOT/*/*`` to ``~/.m2/repository``

.. _maven-repo-backup repository: https://gitlab.com/toccoag/maven-repo-backup
.. _built-in import and export functionality: https://www.jfrog.com/confluence/display/RTF/Importing+and+Exporting#ImportingandExporting-RepositoriesImportandExport
