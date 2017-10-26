Remote Debugging of Nice2
=========================

#. forward the debugging port to your machine

    **on OpenShift**:

    .. code:: bash

        oc project toco-nice-${INSTALLATION}
        oc port-forward ${POD_NAME} 40200

    **on app servers** (legacy):

    #. open the **manager.xml** file

        .. code::

            ssh -t tocco@${INSTALLATION}.tocco.ch less /home/tocco/etc/manager/manager.xml

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

        ssh -N -L ${PORT_NUMBER_FROM_MANAGER_XML}:localhost:40200

#. Now Set Up Remote Debugging in IDEA

    .. figure:: remote_debugging/remote_debugging.png
        :scale: 60%

        Add debug configuration in IDEA.
