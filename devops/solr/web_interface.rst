Solr Web Interface
==================

Connecting to the Web Interface (Manged Server)
-----------------------------------------------

This section describes how to access the Solr web interface, if Solr
is running on a managed server. This is the case when the solr URL is
``https://*.tocco.cust.vshn.net:8983/…``::

    $ oc set env --list dc/nice | grep ^NICE2_APP_nice2.enterprisesearch.solrUrl=
    NICE2_APP_nice2.enterprisesearch.solrUrl=https://solr.tocco.cust.vshn.net:8983/solr/nice-tocco

#. Create local SOCKS proxy

   .. parsed-literal::

       ssh -D 3333 **${host_name}**

   **${host_name}** is the host name that appeared in the URL of the output of the command above. In
   the example above, it is *solr.tocco.cust.vshn.net*.

#. Configure Socks in Firefox

   #. enter ``about:preferences`` in URL bar
   #. find *Network Proxy* → *Settings…*
   #. select *Manual proxy configuration*
   #. enter *localhost* as *SOCKS Host* and *3333* as *Port*
   #. check *Proxy DNS when using SOCKS v5*

#. Obtain the password

   Use the password of user *tocco* that you can find in the file `infrastructure/solr.yml`_
   within VSHN's hieradata Git repository.

#. Login

   Visit https\://\ **${host_name}** in the browser.

   .. figure:: resources/solr_dashboard.png
      :scale: 60%

      Solr's web interface

#. Undo the proxy settings in Firefox


Connecting to the Web Interface (Pod)
-------------------------------------

This section describes how to access the Solr web interface, if Solr
is running as seperate Pod. This is the case when the solr url is
``http://solr:8983/…``::

    $ oc set env --list dc/nice | grep ^NICE2_APP_nice2.enterprisesearch.solrUrl=
    NICE2_APP_nice2.enterprisesearch.solrUrl=http://solr:8983/solr/nice2_index

#. Switch to the right project

    .. parsed-literal::

        $ oc project toco-nice-**${INSTALLATION}**

#. Find the Solr pod

    .. parsed-literal::

        $ oc get pods -l app=solr -o name
        pods/**solr-23-3r3m8**

#. Forward Solr's port

    Forwards the port 8983 in container to port 8080 on your local machine.

    .. parsed-literal::

        $ oc port-forward **solr-23-3r3m8** 8080:8983

#. Connect to it using any browser

    Connect to ``http://localhost:8080``

    .. figure:: resources/solr_dashboard.png
        :scale: 60%

        Solr's web interface


.. _infrastructure/solr.yml: https://git.vshn.net/tocco/tocco_hieradata/blob/master/infrastructure/solr.yaml
