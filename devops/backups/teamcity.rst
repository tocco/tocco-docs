Teamcity Backups
================

Automatic Backups During Updates
================================

The :ansible-repo-file:`Ansible playbook to install updates <playbooks/install_updates/main.yml>`
creates a backup whenever TeamCity is updated. The updates are stored in
``/home/tocco/.BuildServer/backup`` on the TeamCity server and kept for 30 days.

It is also possible to `view backups in TeamCity`_ itself.


.. _view backups in TeamCity: https://tc.tocco.ch/admin/admin.html?item=backup&tab=backupHistory.
