Nice2 Build Errors
==================

Maven Repository Delivers Malformed Files
-----------------------------------------

Error :blue:`A`: Corrupted Jar
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Cause:

    .. code::

        error in opening zip file

Full message:

    .. parsed-literal::

        [INFO] -------------------------------------------------------------
        [ERROR] COMPILATION ERROR :
        [INFO] -------------------------------------------------------------
        [ERROR] error reading /home/tocco/.m2/repository/ch/tocco/manager/tocco-manager-alerts/1.2-SNAPSHOT/**tocco-manager-alerts**-1.2-SNAPSHOT.jar; **error in opening zip file**
        [ERROR] error reading /home/tocco/.m2/repository/ch/tocco/manager/tocco-manager-alerts/1.2-SNAPSHOT/**tocco-manager-alerts**-1.2-SNAPSHOT.jar; **error in opening zip file**
        [ERROR] /build/agent/work/74cb698015a78e5e/monitoring/api/src/main/java/ch/tocco/nice2/monitoring/AlertPublisher.java:[3,31] package ch.tocco.manager.alerts does not exist
        [ERROR] /build/agent/work/74cb698015a78e5e/monitoring/api/src/main/java/ch/tocco/nice2/monitoring/AlertPublisher.java:[11,25] cannot find symbol
          symbol:   class Alert
          location: interface ch.tocco.nice2.monitoring.AlertPublisher
        [ERROR] /build/agent/work/74cb698015a78e5e/monitoring/api/src/main/java/ch/tocco/nice2/monitoring/AlertPublisher.java:[12,26] cannot find symbol
          symbol:   class Alert
          location: interface ch.tocco.nice2.monitoring.AlertPublisher


Error :green:`B`: Corrupted Pom
```````````````````````````````

Cause:

    .. code::

        An invalid XML character (Unicode: 0xb) was found in the value of attribute "title" and element is "article".

Full message:

.. parsed-literal::

    [Step 7/10] [INFO] ------------------------------------------------------------------------
    [Step 7/10] [INFO] Building nice2-optional-apprenticeshipcorrespondence-module 1.0-SNAPSHOT
    [Step 7/10] [INFO] ------------------------------------------------------------------------
    [Step 7/10] Importing data from '/build/agent/work/59745ac4296dbd52/optional/apprenticeshipcorrespondence/module/target/failsafe-reports/TEST-\*.xml' (not existing file) with 'surefire' processor
            [ch.tocco.nice2.optional.apprenticeshipcorrespondence:nice2-optional-apprenticeshipcorrespondence-module] **[Fatal Error] :1:25: An invalid XML character (Unicode: 0xb) was found in the value of attribute "title" and element is "article".**
            [ch.tocco.nice2.optional.apprenticeshipcorrespondence:nice2-optional-apprenticeshipcorrespondence-module] ##teamcity[importData tc:tags='tc:internal' type='surefire' path='/build/agent/work/59745ac4296dbd52/optional/apprenticeshipcorrespondence/module/target/surefire-reports/TEST-\*.xml' whenNoDataPublished='nothing' logAsInternal='true']


Cause
`````

In both cases, :blue:`A` and :green:`B`, a file received from :term:`Artifactory` is corrupted. In case of a ``*.jar``,
the resulting error is en extraction error (:blue:`A`) and in case of a ``pom.xml`` a parser error (:green:`B`).

The corruption is usually caused by and :term:`Remote Repository` used by our :term:`Artifactory`. The latest incident
happened when one of the repositories was moving and for that time served "Project web is currently offline pending the
final migration of its data to our new datacenter" for all requests. To check if and how the files are
corrupted take a look at the affected file in the local cache (``~/.m2/repository/``) or on said :term:`Artifactory`.

Solution
````````

#. Find affected repository

    .. figure:: resources/artifactory_search.png
        :scale: 60%

        Artifactory Search

    1. Search for the affected package, in the case of :blue:`A`, ``tocco-manager-alerts``. If you see the error shown
       in :green:`B`, which doesn't show a package name, scan the rest of the output for the error ``error in opening
       zip file``.

    2. Sort by modification date; the corruption likely occurred within the past few hours.

    3. Show the first corrupted file listed in the Repository Browser.

#. Ensure that corrupted files are no longer fetched from the affected :term:`Remote Repository`.

    .. figure:: resources/artifactory_browser.png
        :scale: 60%

        Artifactory Repository Browser

    In the Repository Browser, you should be able to figure out what :blue:`repository` is affected. Once you know, go
    to **Admin** → **Remote** → **${AFFECTED_REPOSITORY}** in the settings.

    In the remote setting you have two option to ensure that corrupted packages are no longer fetched:

        a) Set the repository offline. This will work if all needed files are in the cache which should be the case.
        b) Set an include pattern (and don't forget to remove the default ``**/*``). This is what I did last time, when
           the this issue occurred, for the ``jasperreports.sourceforge.net`` repository [#f1]_.

#. Remove corrupted files

    Now that it is ensured that no more corrupted files are fetched, go back to the Repository Browser and remove the
    corrupted files. The corrupted files need to be removed from your local cache also (``rm -rf ~/.m2/repository/``).

#. Clean up

    Once the :term:`Remote Repository` is working properly again, make sure you set the repository online again.


.. rubric:: Footnotes

.. [#f1] By default, the include pattern ``**/*`` is set which will try to use the repository for all packages, even
         ``ch.tocco.…`` packages. By making sure an appropriate pattern is set, you can make sure :term:`Artifactory`
         doesn't accidentally fetch packages from there should a :term:`Remote Repository` fail to report the package
         absent.
