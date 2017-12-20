Servers And Services
====================

Internal Servers
----------------

The services we need for the daily business are all running on the four host on the first entree sol.
On these servers run services like: Gerrit, Postgres and Sonar.
A more precise list of what server runs which services can be found `below <#provided-services>`_.
All these servers run :doc:`Proxmox <proxmox>` and can be accessed under the following addresses.
Credentials for the Proxmox hosts: User: root Password: root password.

        #. host03a.tocco.ch

        #. host03b.tocco.ch

        #. host03c.tocco.ch

        #. host03d.tocco.ch

Also on the first entree sol you'll find the telephone system.
It is reachable under the following domain:

        #. tel.tocco.ch


Provided Services
-----------------

The following table shows what servers runs which service. Some of the services are passive.
That means you do not use them active, e.g. The Teamcity-Agents.

   ================== ================== ================== ==================
    host03a             host03b           host03c           host03d
   ================== ================== ================== ==================
    Maven               JAVA-WS           Postgres          TC-Agent-6
    File server         Edge              TC-Agent-4        TC-Agent-7
    Nagios              Kibana                              TC-Agent-8
    TC-Agent-1
    Gerrit/Git
    Teamcity
    Wiki
   ================== ================== ================== ==================

