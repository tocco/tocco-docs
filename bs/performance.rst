######################
Unreliable Application
######################

.. warning::

   This document is work in progress. Expect to find errors.

Slowness
========

This information should be gathered first:

* What is slow?

  * Does it only affect a certain entity or selection? Which one(s)?
* Is it slow consistently or sporadically?
* Has it been faster in the past.
* If possible, exact steps to reproduce.


Connectivity Issues (Application not available)
===============================================

TODO: thread/memory dumps, analyzing logs

#. This information should be gathered first:

   * Who is expiriencing the issue? (one persion, serveral, all, only people at one location, etc.)
   * Where does the issue appear? (intranet, extranet, specific URL, etc.)
   * How does the issue materialize? (particular error message, applicaiton not available, etc.)
   * What network is affected? (school network, city network)
   * What client was used:

     * OS (incl. version)
     * Browser (incl. version)
     * Proxies
   * Is there a way to trigger the issue? Are there any steps that need to be taken to reproduce
     the issue?

#. If the issue cannot be reproduced, ask the customer to `create an HAR file`_.

#. Then check if there have been restarts:

   This is often caused by out-of-memory errors.

   .. parsed-literal::

       $ oc get pods
       NAME               READY     STATUS              RESTARTS   AGE
       nice-69-kvjlf      2/2       Running             **3**          7d

  If you see restarts, check the log for an *OutOfMemoryError* before the restart::

      $ oc logs -c nice nice-69-kvjlf | n2log-unscramble 'OutOfMemoryError'
      Terminating due to java.lang.OutOfMemoryError: Java heap space

  If not, check for *Thread starvation* messages::

      $ oc logs -c nice nice-69-kvjlf | n2log-unscramble 'Thread starvation'
      2020-05-12 06:22:58.983 WARN  com.zaxxer.hikari.pool.HikariPool [HikariPool-1 housekeeper]
      HikariPool-1 - Thread starvation or clock leap detected (housekeeper delta=1m9s640ms987µs196ns)

  Technical note:

      *Thread starvation* often happens because the GC threads are using all the resources. This
      usually happens shortly before an OutOfMemoryError.

  .. todo::

     What should be done if application ran out of memory?

#. Check for unusual activities in logs::

       $ oc logs -c nice nice-69-kvjlf | n2log-unscramble

   Warnings and errors only::

       $ oc logs -c nice nice-69-kvjlf | n2log-unscramble -l warn

#. Check for unusual events::

    $ oc describe pod nice-69-kvjlf | grep -A 999 Events:$

    TODO: For what does one look?

    Note: Liveness and Readiness probe failure are expected during application start.


Failing or Slow Logins
======================

TODO (detecting REST use without *nice_auth* cookie)

* In <2.25, this is frequently caused by using the REST API without *nice_auth* cookie. Check
  for frequent logins::

      $ oc logs -c nice nice-69-kvjlf | n2log-unscramble AuthenticationHandler
      ====================================================================================================
      2020-05-20 12:28:26 INFO  - thread: qtp1544300373-17493, logger: nice2.userbase.DbAuthenticationHandler

      Successful Login: Principal[PK:5020, username:rretep@tocco.ch] Session[PK:460890] IP:38.175.164.17
      ====================================================================================================
      2020-05-20 12:39:32 INFO  - thread: qtp1544300373-17551, logger: nice2.userbase.DbAuthenticationHandler

      Successful Login: Principal[PK:4478, username:data-import] Session[PK:460891] IP:52.127.123.220
      ====================================================================================================
      …

  Look for a high number of logins done using a single login. Also, look for login indicating a non-human
  client like *data-import* in the above example. If this happens please inform the customer that the
  *nice_auth* cookie needs to be set `acoording to documentation <nice_auth cookie>`_ (section *nice_auth*).

  Should this prevent user from logging in, consider deactivating the login temporarily. Worst
  case, deactivate it via SQL:

  .. code-block:: sql

      UPDATE nice_principal
        SET fk_principal_status = (SELECT pk FROM nice_principal_status WHERE unique_id = 'inactive')
        WHERE username = '${USERNAME}';

.. _create an HAR file: https://support.zendesk.com/hc/en-us/articles/204410413-Generating-a-HAR-file-for-troubleshooting
.. _nice_auth cookie: https://test224.tocco.ch/nice2/swagger
