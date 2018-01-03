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

#. Show all available backups.

   .. code:: 

      sudo burp -a list 

      Backup: 0000054 2017-10-21 23:08:28 +0200 (deletable)
      Backup: 0000055 2017-10-22 22:33:40 +0200 (deletable)
      ...

#. Show content of a backup.

   .. code::

      sudo burp -a list -b 0000055

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

      sudo burp -a list -b 55 -r 'nice2'

      Backup: 0000055 2017-10-22 22:33:40 +0200 (deletable)
      With regex: nice2
      /home/nils.kreienbuehl/dumps/nice2_ethz_dump_cloud.sql
      /home/nils.kreienbuehl/dumps/nice2_heks.sql
      /home/nils.kreienbuehl/dumps/nice2_heks2017-09-13T12:20:29Z.sql
      ...

#. Restore all files in a backup that match the given regex condition.

   .. code::

      sudo burp -a restore -b 55 -r 'nice2'
      
      2017-10-23 18:02:38 +0200: burp[12429] doing restore 55:nice2
      2017-10-23 18:02:38 +0200: burp[12429] doing restore confirmed
      ...
      2017-10-23 18:02:39 +0200: burp[12429] restore finished

#. We saw how to list and restore backups with burp. But you are not bound to you actual client. With burp you can even list and restore from other Clients. This is very easily done with the '-C' option.

  .. code::

     (suppose you are on db1.tocco.cust.vshn.net)

     sudo burp -a list -C db2.tocco.cust.vshn.net

     2018-01-03 10:26:07 +0100: burp[17912] Switched to client db2.tocco.cust.vshn.net
     ...
     Backup: 0000106 2017-12-11 01:26:02 +0100 (deletable)
     Backup: 0000113 2017-12-18 01:12:10 +0100 (deletable)
     Backup: 0000120 2017-12-25 01:07:33 +0100 (deletable)
     ...
     2018-01-03 10:26:07 +0100: burp[17912] List finished ok
