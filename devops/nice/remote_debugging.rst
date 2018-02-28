Remote Debugging of Nice2
=========================

#. forward the debugging port to your machine

    **on OpenShift**:

    #. switch project

        .. parsed-literal::

            $ oc project toco-nice-**${INSTALLATION}**

    #. find running pod

        .. parsed-literal::

            $ oc get pods -l run=nice --show-all=false
            NAME           READY     STATUS    RESTARTS   AGE
            **nice-3-2nl1q**   2/2       Running   0          49m


    #. forward port

        .. parsed-literal::

            $ oc port-forward **nice-3-2nl1q** 40200

    **on app servers** (legacy):

    #. open the **manager.xml** file

        .. code::

            ssh -t tocco@${INSTALLATION}.tocco.ch less /home/tocco/manager/etc/manager.xml

    #. find the ``<debug port="PORT_NUMBER">`` of **CUSTOMER**

        .. parsed-literal::

            <java name="nice2-**CUSTOMER**" extends="nice2">
              <arg>-Dch.tocco.nice2.runenv=test</arg>
              <user>koshiatsu_user</user>
              <property name="nice2.home">${tocco.home}/nice2/koshiatsutest</property>
              <app-arg>jmxmp.port=20319</app-arg>
              <property name="java.home">/usr/lib/jvm/tocco-java-8</property>
              <debug port="**36048**"/>
            </java>

    #. now forward the port to your machine

        ssh -N -L 40200:localhost:${PORT_NUMBER_FROM_MANAGER_XML} ${INSTALLATION_NAME}.tocco.ch

#. Now Set Up Remote Debugging in IDEA

    .. figure:: remote_debugging/remote_debugging.png
        :scale: 60%

        Add debug configuration in IDEA.
