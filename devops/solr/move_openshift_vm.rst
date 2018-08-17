=================================================
Move Solr Core from OpenShift to a Managed Server
=================================================

#. Add :term:`core <Solr core>` to Puppet config

   Add a new core to section ``profile_solr::hiera_cores`` in the `Puppet configuration <https://git.vshn.net/tocco/tocco_hieradata/blob/master/infrastructure/solr.yaml>`_.

   .. hint::

      Cores names follow this schema: ``nice-${INSTALLATION}``.

   .. warning::

      Configuration changes take up to 30 minutes to become active.

#. Switch project

    .. parsed-literal::

        oc project toco-nice-**${INSTALLATION}**

#. Backup index data

    .. code::

       oc exec $(oc get pods -l app=solr -o name | sed 's/.*\///') -- bash -c 'mkdir /tmp/index && curl -s "http://localhost:8983/solr/nice2_index/replication?command=backup&location=/tmp/index&name=snapshot"'

    Verify that the output contains ``"status": "OK"``.

#. Check if backup succeeded

    .. code::

        oc exec $(oc get pods -l app=solr -o name | sed 's/.*\///') -- bash -c 'curl -s "http://localhost:8983/solr/nice2_index/replication?command=details"'

    You should see something like this::

        …
        "backup":[
          "startTime","Thu Aug 16 12:13:57 UTC 2018",
          "fileCount",199,
          "status","success",
          "snapshotCompletedAt","Thu Aug 16 12:14:48 UTC 2018",
          "snapshotName","snapshot"]}}

    .. note::

        Backup is done asynchronously. Hence, the backup may be in progress still.

#. Copy index out of pod

    .. code::

       oc exec $(oc get pods -l app=solr -o name | sed 's/.*\///') -- tar -C /tmp/index -cz . >index.tar.gz

#. Copy index to managed server

    .. parsed-literal::

        scp index.tar.gz solr\ **${N}**.tocco.cust.vshn.net:

    .. note::

        Servers are numbered and thus you'll have to replace **${N}** with the appropriate number (e.g. solr1.tocco.cust.vshn.net).

#. Restore index

    .. parsed-literal::

        ssh solr\ **${N}**.tocco.cust.vshn.net
        mkdir index
        cd index
        tar xf ../index.tar.gz
        curl --insecure -s "https\://localhost:8983/solr/nice-**${INSTALLATION}**/replication?command=restore&location=$(pwd)&name=snapshot"

#. Check if restore succeeded

    .. code::

       curl --insecure -s "https://localhost:8983/solr/nice-toccotest/replication?command=restorestatus"

    .. note::

        Restore is done asynchronously. Hence, the restore may be in progress still.


#. Copy configuration

    .. parsed-literal::

        cd /var/lib/solr/data/nice-**${INSTALLATION}**/
        rm -rf conf
        cp -a ../nice-test212/conf .
        echo -e "config=solrconfig.xml\nschema=schema.xml" | tee -a core.properties

#. Reload configuration

    .. code::

       curl --insecure -s "https://localhost:8983/solr/admin/cores?action=RELOAD&core=nice-${INSTALLATION}"

    This should output status code 0::

        {
           "responseHeader":{
           "status":0,
           "QTime":262 }
        }

#. Use new index

    .. parsed-literal::

       oc set env dc/nice -c nice NICE2_APP_nice2__enterprisesearch__solrUrl=https://solr\ **${N}**.tocco.cust.vshn.net:8983/solr/nice-**${INSTALLATION}**

#. Stop Solr on OpenShift

    .. code::

       oc scale --replicas=0 dc/solr

#. Verify Solr is working

    * Search existing entries.
    * Create new entities, see if they are added.
    * Check logs of Nice.

#. Clean up managed server

    .. parsed-literal::

        ssh solr\ **${N}**.tocco.cust.vshn.net
        rm -rf ~/index.tar.gz ~/index

#. Clean up OpenShift

    .. code::

       oc delete dc solr
       oc delete pvc solr
