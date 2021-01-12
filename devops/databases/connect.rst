###################
Connect to Database
###################

.. _connect-to-db-via-openshift:

Option a) via OpenShift
=======================

Connect to the database used by the current project::

    $ oc rsh -c nice dc/nice psql
    psql (9.6.20, server 9.5.24)

    nice_master=>


.. note::

   Use option b) if you require superuser privileges.


Option b) via ssh
=================

#. Identify DB server:

   Look for *db_server*  in *${ANSIBLE_REPO}/tocco/config.yml*. This
   is the DB server used by an installation.

#. Find password of user postgres::

       cd ${ANSIBLE_REPO}/tocco
       ansible-vault view secrets2.yml | grep ${DB_SERVER}

#. ssh into machine::

       $ ssh ${DB_SERVER}

#. connect to server::

       $ psql -h localhost -U postgres
