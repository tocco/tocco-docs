Thread and Memory Dumps (App Server)
====================================

Creating a Thread Dump
----------------------

Create the thread dump:

    mgrsh thread-dump nice2-${INSTALLATION}

The thread dumps itself can be found in the log at ``~tocco/nice2/${INSTALLATION}/var/log/nice.log``.

.. tip::

    ``mgrsh thread-dump`` sometimes fails with *Daemon with name 'xxx' is not running* despite
    the installation being running. In such a case, manually send SIGQUIT to the process as
    detailed below.

    .. important::

        Root access required.

    Find the Nice process:

    .. parsed-literal::

        $ sudo lsof ~/tocconice2/ecap/var/log/nice.log
        COMMAND  PID      USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
        java    **5851** ecap_user  509w   REG  252,3  4511813 1703988 /home/tocco/nice2/ecap/var/log/nice.log

    Send SIGQUIT to the process:

    .. parsed-literal::

        $ sudo kill -SIGQUIT **${PID_FROM_LSOF_OUTPUT}**

    The thread dump can now be found in the log file mentioned above.


Creating a Memory Dump Manually
-------------------------------

    .. important::

        Root access required.

    Find the Nice process and user:

    .. parsed-literal::

        $ sudo lsof ~/tocconice2/ecap/var/log/nice.log
        COMMAND  PID      USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
        java    **5851** **ecap_user**  509w   REG  252,3  4511813 1703988 /home/tocco/nice2/ecap/var/log/nice.log

    Create memory dump:

    .. parsed-literal::

        $ path=$(sudo -u **${USER_FROM_LSOF_OUTPUT}** mktemp -d)
        $ sudo -u **${USER_FROM_LSOF_OUTPUT}** jmap -dump:format=b,file="$path/dump" **${PID_FROM_LSOF_OUTPUT}**
        Dumping heap to **/tmp/tmp.Blbd4xcqdz/dump** ...
        Heap dump file created


Creating Memory Dump on OOM
---------------------------

    .. important::

        Root access required.

Memory dumps are created automatically and stored as ``~tocco/manager/var/daemonsnice2-${INSTALLATION}/*.hprof``.
