Postgres Replication
====================

What to do if Replication Streaming on Postgres fails
-----------------------------------------------------

.. _read_first: 

Important! Read this section first before doing anything!
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this section, very powerful commands are described. So before execute anything, be 100% sure that you are on the right server!
Else you could cause a lot of damage. 

To check if you are on the right server use the following query.
On master the result is empty. On slave it is not (see example).

#. Check if you are on slave

   .. code::

       SELECT pg_last_xlog_receive_location();

         pg_last_xlog_receive_location
        -------------------------------
                2493/54AAF118


Check if Replication is working
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First of all you should check the delay. If the delay is critical nagios or Backoffice will inform you about it.
This will look like this:

#. Check the delay

   .. code::

      POSTGRES_HOT_STANDBY_DELAY CRITICAL: DB postgres (host:db01slave) 147866242312 seconds

   If the number (delay in seconds) is extremely high then you know something must be wrong with the replication.

#. Another way to check the delay directly is

   .. code::

      $ ssh db01a

      $ psql -U postgres -h db01slave

      SELECT now() - pg_last_xact_replay_timestamp() AS delay;

               delay
           --------------
           04:15:55.03134

   It prints the delay in human readable time format like (hh:mm:ss). 
   The advantage of this query is, that it is very comprehensive. So you don't have to calculate a lot, the output is appropriate.
   If the delay is very small the slave is probably reading from master at that moment. But always have an look at the issue.

#. Check the last replay

   .. code::

      SELECT pg_last_xact_replay_timestamp();

        pg_last_xact_replay_timestamp
       -------------------------------
        2017-10-10 11:45:00.945915+02

   It prints the timestamp for the last replication. It is more comprehensive than just a number.
   If the delay is more than 3 hours, you should have a look at the postgres log file: /var/log/postgres/postgresql-9.5-main.log.
   If you find messages like "0000000200002440000000A8: file already removed" then you have to do the basebackup.
   Otherwise you could have a look at the postgres configurations. If you can't see anything odd, a restart of postgres could solve the problem.

.. hint:: 
   Postgres doesn't replay any changes during dumps.

#. Restart Postgres-Slave

   .. code::

      $ sudo systemctl restart postgresql@9.5-main.service


Basebackup
^^^^^^^^^^

:ref:`Important! <read_first>`
All these commands must only be executed on de Slave (db01slave)!
The initial user for this procedure should be tadm.

#. save password from /postgres/postgres_data/recovery.conf
#. change to user root
#. copy the password

   .. code::

      sudo su

      cat /postgres/postgres_data/recovery.conf

#. Base backup

   .. code::

      screen -S restore_slavei

      cp -a /etc/postgresql/9.5/main/ /postgres/backup/main

      pg_dropcluster --stop 9.5 main

      cd /etc/postgres

      mkdir 9.5

      chown -R postgres:postgres 9.5

      cp -a /postgres/backup/main /etc/postgresql/9.5/main

      ^D or exit

      sudo su postgres

      pg_basebackup -h db01master -U pg_replica -D /postgres/postgres_data/ -v -P --xlog-method=stream

      cd /postgres/postgres_data/ && mv recovery.done recovery.conf

      recovery.conf change (Slave-IP) -> (Master-IP)

      ^D or exit

      sudo systemctl start postgresql@9.5-main.service

