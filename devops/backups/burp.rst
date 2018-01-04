BURP - Backup And Resotre Program
=================================

What is BURP?
-------------

So what is BURP and why do we use it?

BURP is a network backup and restore program. It attempts to reduce network traffic and the amount of space that is used by each backup. - `burp.grke.org <http://burp.grke.org>`_

BURP basically makes a backup of the whole file system. This dumps are very neat, you can view or restore the whole backup or just a single file. 
Currently the backups on are made with a lot of different scripts. 
The standard procedure of the scripts is something like that:

#. Compress a directory tree.

#. Move the compressed directories to the backup server (rsync).

#. Delete the compress files if they are older than x days.

For every step we have a single script and sometimes there is a collection of script for every server.

BURP combines all these steps in one Program and makes backups more comprehensible and easy to handle. 

Search and restore a file
-------------------------
#. Login to a host via ssh

   .. code::
      
      ssh db1.tocco.cust.vshn.net

#. Show all available backups.

   .. note::
      
      With the -C option you can list/restore backups from another Client. It is also possible to use the own Client, for that just don't use the -C option.

   .. code:: 

      sudo burp -C db2.tocco.cust.vshn.net -a list 

      Backup: 0000054 2017-10-21 23:08:28 +0200 (deletable)
      Backup: 0000055 2017-10-22 22:33:40 +0200 (deletable)
      ...

#. Show content of a backup.

   .. code::

      sudo burp -C db2.tocco.cust.vshn.net -a list -b 0000055

      /var/spool/postfix/public/showq
      /var/spool/postfix/saved
      /var/spool/postfix/trace
      /var/spool/postfix/usr
      ...

   .. note:: 
      | the number 0000055 refers to the number of the backup in step 1.
      | e.g. Backup: 0000055 2017-10-22 22:33:40 +0200 (deletable)

#. Search for all files in backup that match the regex.

   .. code::

      sudo burp -C db2.tocco.cust.vshn.net -a list -b 55 -r '/var/lib/postgresql-backup/postgres-*'

      Backup: 0000055 2017-10-22 22:33:40 +0200 (deletable)
      With regex: postgresql-backup
      /var/lib/postgresql-backup/postgres-nice_create_installation.dump.gz
      /var/lib/postgresql-backup/postgres-nice_ethz.dump.gz
      /var/lib/postgresql-backup/postgres-nice_heks.dump.gz
      ...

#. Restore all files in a backup that match the given regex.

   .. code::

      sudo burp -C db2.tocco.cust.vshn.net -a restore -b 55 -r '/var/lib/postgresql-backup/postgres-*'
      
      2017-10-23 18:02:38 +0200: burp[12429] doing restore 55:/var/lib/postgresql-backup/postgres-*
      2017-10-23 18:02:38 +0200: burp[12429] doing restore confirmed
      ...
      2017-10-23 18:02:39 +0200: burp[12429] restore finished
