Firewall (ZyWALL USG 100)
=========================

You can find the Firewall in the rack located at the kitchen in the left back corner. It should be above the Router.

Access
------

You can access the firewall under **https://10.27.1.1**. The credentials can be found in the Backoffice. Just search for firewall in the entity server.

Configuration Backup
--------------------

In case you need a backup of the firewall configuration files for a restore, a backup can be found on backup01.tocco.ch.

   .. parsed-literal::
   
      $ ssh tadm@backup01.tocco.ch

      $ cd /backup/firewall

      $ ls -al ->
   
      drwxr-xr-x  2 root root  4096 Mar 22 12:43 .
      drwxr-xr-x 10 root root  4096 Mar 22 12:41 ..
      -rw-r--r--  1 tadm tadm 19685 Mar 22 12:43 **zywall-usg-100-config.tar.gz**
   
