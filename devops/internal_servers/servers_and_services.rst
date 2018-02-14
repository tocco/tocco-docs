Servers And Services
====================

Internal Servers
----------------

The services we need for the daily business are all running on the four host on the first entreesol.
On these servers run services like: Gerrit, Postgres and Sonar.
A more precise list of what server runs which services can be found `below <#provided-services>`_.
All these servers run :doc:`Proxmox <proxmox>` and can be accessed under the following addresses.
Credentials for the Proxmox host: User: root Password: root password.

        #. host03a.tocco.ch

        #. host03b.tocco.ch

        #. host03c.tocco.ch

        #. host03d.tocco.ch

Also on the first entreesol you'll find the telephone system.
It is reachable under the following domain:

        #. tel.tocco.ch


Provided Services
-----------------

The following table shows what server runs which service. Some of the services are passive.
That means you do not use them active, e.g. The Teamcity-Agents.

   ================== ================== ================== ==================
    host03a             host03b           host03c           host03d
   ================== ================== ================== ==================
    Maven (100)         JAVA-WS (100)     Postgres (111)    TC-Agent-6 (102)
    File server (101)   Edge (101)        TC-Agent-4 (116)  TC-Agent-7 (2007)
    Nagios (107)        Kibana (118)                        TC-Agent-8 (2008)
    TC-Agent-1 (108)
    Gerrit/Git (109)
    Teamcity (115)
    Wiki (1101)
   ================== ================== ================== ==================

