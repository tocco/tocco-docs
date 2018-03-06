Thread and Memory Dumps
=======================

Creating a Thread Dump
----------------------

#. find pod

    .. parsed-literal::

        $ oc get pods -l run=nice --show-all=false
        NAME           READY     STATUS    RESTARTS   AGE
        **nice-3-2nl1q**   2/2       Running   0          49m

#. create thread dump

    .. parsed-literal::

        $ oc exec -c nice **nice-3-2nl1q** -- kill -SIGQUIT 1

#. obtain thread dump

    .. parsed-literal::

        $ oc logs -c nice **nice-3-2nl1q**
        Full thread dump OpenJDK 64-Bit Server VM (25.151-b12 mixed mode):

        "pool-7-thread-1182" #4356 prio=5 os_prio=0 tid=0x00007f9b2c040000 nid=0x12ec waiting on condition [0x00007f9a765fa000]
          java.lang.Thread.State: TIMED_WAITING (parking)
        …


Creating a Memory Dump Manually
-------------------------------

#. find pod

    .. parsed-literal::

        $ oc get pods -l run=nice --show-all=false
        NAME           READY     STATUS    RESTARTS   AGE
        **nice-3-2nl1q**   2/2       Running   0          49m

#. create dump

    .. parsed-literal::

        $ oc exec -c nice **nice-3-2nl1q** -- jmap -dump:format=b,file=/app/var/heap_dumps/forced_dump.hprof 1
        Dumping heap to /app/var/heap_dumps/forced_dump.hprof ...
        Heap dump file created

#. copy dump

    .. parsed-literal::

        $ oc cp -c nice **nice-3-2nl1q**:/app/var/heap_dumps/forced_dump.hprof forced_dump.hprof
        $ ls -lh forced_dump.hprof
        -rw-r--r-- 1 user user 2.3G Mar  6 13:55 forced_dump.hprof

Creating Memory Dump on OOM
---------------------------

#. enable dumps on OOM

    .. parsed-literal::

        oc set env dc/nice NICE2_DUMP_ON_OOM=true

    .. warning::

        This will restart Nice automatically!

#. add persistent volume for dumps

   Create an appropriately sized volume for ``/app/var/heap_dumps`` as described in :ref:`persistent-volume-creation`.

#. wait for OOM crash

#. find a running pod

    .. parsed-literal::

        $ oc get pods -l run=nice --show-all=false
        NAME           READY     STATUS    RESTARTS   AGE
        **nice-3-2nl1q**   2/2       Running   0          49m

#. find the dump you want

    .. parsed-literal::

        $ oc exec -c nice **nice-3-2nl1q** -- ls -lh /app/var/heap_dumps/
        -rw-r--r-- 1 peter peter 1.2G Mar  6 13:57 **memory-dump-asfkosfnkt.hprof**

#. copy the dump

    .. parsed-literal::

        $ oc cp -c nice **nice-3-2nl1q**:/app/var/heap_dumps/**memory-dump-asfkosfnkt.hprof** dump.hprof
        $ ls -lh dump.hprof
        -rw-r--r-- 1 user user 1.2G Mar  6 14:00 dump.hprof

#. disable dumps on OOM

    .. code::

        oc set env dc/nice NICE2_DUMP_ON_OOM-

    .. warning::

        This will restart Nice automatically!

#. remove persistent volume for dumps

    Remove previously created persistent volume for ``/app/var/heap_dumps`` as outlined in
    :ref:`persistent-volume-removal`.
