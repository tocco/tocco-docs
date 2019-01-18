IDEA/Maven Troubleshooting
==========================

Idea Performance
----------------

If your Idea isn't running smoothly or becomes unresponsive when you open multiple projects, try increasing memory like this:

**Help** → **Edit Custom VM Options…** and then set max. memory higher (**-Xmx…**). Don't forget to restart Idea


Failing Maven Import in Idea
----------------------------

Check Log File
``````````````
**Help** → **Show log in …** and then open ``idea.log``.


Increase Memory
```````````````

It happens frequently that Maven projects are marked red in IDEA after a Maven import and you have to retry the import
multiple times. This often happen because there isn't enough memory.

Instructions:

    Go to **File** → **Settings** → **Build, Execution, Deployment** → **Buid Tools** → **Maven** →
    **Importing** → **VM options for Importer** and set **-Xmx…** higher.

Decrease Number of Parallel Jobs
````````````````````````````````

If you physically don't have enough memory. Consider reducing the number of jobs per CPU.

Instructions:

    **File** → **Settings** → **Build, Execution, Deployment** → **Buid Tools** → **Maven** → **Importing** →
    **Thread**.

    (default 1.5C = 1.5 jobs per CPU)


Failing ``mvn install``
-----------------------

Out of Physical Memory
``````````````````````

If the build keeps failing randomly, often because of failures in Gulp or NPM [#f1]_. In particular, if you see
anything about exit code 137 [#f2]_. Then, it's probably because you ran out-of-memory. Check ``dmesg`` (Linux) for
any message about out of memory. (If there is a large table listing memory usage per process it's an OOM.)

Consider reducing the number of jobs per CPU to reduce memory usage (e.g. by using ``-T1C``).


Exceeding the Alloted Memory
````````````````````````````

If you see **OutOfMemoryError**\ s during the build, it's often because too little memory is allotted to Maven. Consider
increasing the memory limit by setting the environment variable ``MAVEN_OPTS``.

For instance, by adding this in your ``~/.profile`` (and restarting the terminal)::

    export MAVEN_OPTS="-Xmx1024m"

Alternatively, you can decrease the memory usage by decreasing the number of jobs as described in the previous section.

.. _too-many-open-files-maven:

"Too many open files" During Maven Build
````````````````````````````````````````

With the introduction of Java 11, far more files are opened at once during ``mvn build`` sometimes leading to the
error "Too many open files". Increase the the max. number of open files in such a case.

Increasing the limit on Linux:

    Create ``/etc/security/limits.d/open_file_limit.conf`` with this content::

        *                -       nofile          1000000

    **Log out and in again** for this to become effective. Use ``ulimit -n`` to show the current limit.

.. hint::

    Should you have trouble increasing the limit, you can try decreasing the number of parallel jobs running
    during the build using the ``-T`` flag. For instance, specify ``-T3`` to run three jobs or specify ``-T0.5C``
    to run 0.5 jobs per CPU core.

.. rubric:: Footnotes

.. [#f1] Because the out-of-memory killer on Linux sacrifices children first. Hence Gulp and NPM tend to be killed
         first. Other OSes behave differently and thus symptoms may differ (e.g. you may see memory allocation errors
         instead).
.. [#f2] Exit statuses higher than 128 are usually 128 + SIGNAL_NUMBER (see ``kill -l``). In this case 128 + 9 (SIGKILL)
         = 137.
