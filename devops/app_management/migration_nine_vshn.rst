Migration Nine → VSHN
=====================

Pre-Move
--------

* Custom Nginx config

* Forward ACME challenges to VSHN

  In all *server { … }* blocks for the installation, replace this::

      include /etc/nginx/include/certbot.conf;

  with::

      include /etc/nginx/include/certbot-vshn.conf;

  (See *find the right config file* below if you're unsure about
  how to find the *server { … }* blocks.)

  This proxies the */.well-known/acme-challenges/* path to VSHN which
  allows issuing TLS certificates there. As a result, certificates
  can no longer be issued at Nine. However, existing certificates will
  remain valid for at least another 25 days.

* Reload Nginx configuration::

      systemctl reload nginx

* Update *config.yml*:

  * Get hostnames from */etc/nginx/sites-enabled/\*.conf*

    Find the right config file::

        grep ${INSTALLATION} /etc/nginx/sites-enabled/*.conf

    Get the host names / routes from the *server_name* directive. **There
    may be multiple server { … } blocks and thus multiple server_name directives
    to consider.**

    In case a regular expression is used in *server_name*, you can read the
    host names from the TLS cerificate specified via *ssl_certificate*::

        sudo openssl x509 -in ${PATH_TO_CERT} -text |grep -A 1 'Subject Alternative Name'

    In case a regex is used, also check domains in CMS to verify nothing was missed::

        SELECT DISTINCT substring(url from '^https?://([^/]+)') AS url
        FROM (
            SELECT unnest(regexp_split_to_array(alias, E'\n')) AS url
            FROM nice_domain
            WHERE alias <> ''
            UNION SELECT url
            FROM nice_domain
            WHERE url <> ''
        ) AS sub
        ORDER BY 1;

  * Remove ``location: nine``

  * Remove ``app_server``

  * Add ``solr_server`` if missing

    Recreating the Solr index can take a long time for large indexes. In such cases,
    copy the index from the app server to the Solr server. In case you're unsure how
    long reindexing takes, measure the time when migrating the test system and then
    decide whether production can be reindex or if the index has to be copied.

    If you want to copy the index, use the backup functionality:

    On app server::

        # start backup
        curl -u tocco -p -s "https://${INSTALLATION}.tocco.ch/nice2/solr/nice2_index/replication?command=backup&location=/home/tocco/nice2/${INSTALLATION}/var/solr_index_backup&name=snapshot"

        # check status
        curl -u tocco -p -s "https://${INSTALLATION}.tocco.ch/nice2/solr/nice2_index/replication?command=details"

    This creates a backup at */home/tocco/nice2/${INSTALLATION}/var/solr_index_backup*. See
    :doc:`/devops/solr/move_openshift_vm` starting with *Check if backup succeeded* for the
    rest of the instructions.

  * Custom application.local.properties

    Check */home/tocco/nice2/${INSTALLATION}/etc/application.local.properties* on
    app server for customizations. Copy them to the `ansible_properties variable
    <ansible-app-properties>`_ in *config.yml*.

    Some properties in *application.local.properties* are set via Ansible. You generally
    recognize them by having a additional space on both sides of the equal sign (e.g.
    ``property = value`` rather than ``property=value``). Do **not** copy them.

    Non-exhaustive list of properties that should not be copied:

    ================================ ==============================================================
     Property                         Description
    ================================ ==============================================================
     dataSource.\*                    Do not copy. No longer used.
     email.allowedFromDomainsRegex    Do not copy. Set by Ansible.
     email.default.from               Do not copy. Set by Ansible.
     email.hostname                   Do not copy. Set by Ansible.
     email.noreply.from               Do not copy. Set by Ansible.
     email.starttls=true              Do not copy. Has been set to true by default for some time.
     hiveapp.http.host                Do not copy. Default must be used on OpenShift.
     hiveapp.http.port                Do not copy. Default must be used on OpenShift.
     nice2.enterprisesearch.\*        Do not copy. Set by Ansible.
     nice2.system.type                Do not copy. No longer used.
     nice2.userbase.captcha.\*        Do not copy. Set by Ansible.
    ================================ ==============================================================

* Setup customer::

      cd ${ANSIBLE_REPO}/root
      ansible-playbook playbook.yml -l ${INSTALLATION} --skip-tags skip_route_dns_verification

  The *skip_route_dns_verificaton* tag skips all DNS checks for the routes. This is required
  since the routes still point to Nine. The configuration change above just forwards
  */.well-known/acme-challenge/* and that's all we need to issue a TLS certificate.

* Verify that TLS certificates have been issued

  List routes with missing certificates::

      oc project toco-nice-${INSTALLATION}
      oc get route -o json | jq '.items[].spec | if .tls | has("key") then empty else .host end'

  Issuing a certificate can take a few minutes. See :ref:`acme-troubleshooting` in case of missing
  certificates.

* LMS:

  Check if LMS module is installed::

      grep -F '.lms<' customer/*/pom.xml

  If it is installed, a :ref:`persistent volume <persistent-volume>` needs to be
  created and the LMS objects located on the app server at
  ``/home/tocco/nice2/${CUSTOMER}/var/lms/`` need to be copied during the
  migration.

* Configure memory

  Get the current configuration from ``/home/tocco/manager/etc/manager.xml`` and
  configure **2.1 times that memory** on OpenShift. See also :ref:`nice-memory`.

  For instance, if memory is set to 1.5 GiB on at Nine, set it to ~3.2 GiB
  at VSHN.

  (On Nine, the configured memory is for Java heap space only, on OpenShift,
  the expected value is the total used memory.)

* Set TTL to 300 seconds for domains managed by us

* Prepare maintenance page

  * Copy template:

    .. parsed-literal::

        cp /etc/nginx/sites-available/000-maintenance-page.template /etc/nginx/sites-available/100-maintenance-page-\ **${INSTALLATION}**\ .conf

  * Adjust page:

    If the customer has a custom maintenance page, reconfigure the *root* directive in the config file. See
    comment in the file itself.

* Prepare nginx redirect to VSHN

  * Copy template:

    .. parsed-literal::

        cp /etc/nginx/sites-available/000-vshn-redirect.template /etc/nginx/sites-available/200-vshn-redirect-\ **${INSTALLATION}**\ .conf

  * Replace the *${ … }* placeholders:

    .. parsed-literal::

        vi /etc/nginx/sites-available/200-vshn-redirect-\ **${INSTALLATION}**\ .conf

    Get **${SERVER_NAMES}**, **${SSL_CERTIFICATE_KEY}** and **${SSL_CERTIFICATE}** from *container\*.conf*.
    If there are multiple *server { … }* blocks referencing different TLS certs, you have to create
    multiple blocks in the config too. Also, note that for every server block in a *container\*.conf*, two
    new blocks have to be created. One for HTTP and one for HTTPS.

* Remove installation from legacy monitoring

  Go to http://monitor01.tocco.ch

  Remove installation:

  * Tab: *Configuration*
  * Sub-tab: *Hosts*
  * In tree on left: *Hosts* → *Websites* → *${INSTALLATION}.tocco.ch* → Details
  * Button: *Delete*

  Apply change:

  * Tab: *Configuration*
  * Sub-tab: *Control*
  * In tree on left: *Commit*
  * Button: *Commit*

  (At this point monitoring at VSHN is setup and will send alerts even while the installation
  is still at Nine.)

* Enable outgoing mails on production system::

      oc set env -c nice dc/nice NICE2_APP_recipientrewrite.enabled=false


Move
----

* Enable maintenance page and redirect:

  .. parsed-literal::

      ln -s ../100-maintenance-page-\ **${INSTALLATION}**\ .conf /etc/nginx/sites-enabled/
      ln -s ../200-vshn-redirect-\ **${INSTALLATION}**\ .conf /etc/nginx/sites-enabled/
      systemctl reload nginx

* Copy DB from Nine to VSHN::

      tadm@db01master.tocco.ch$ sudo -u postgres pg_dump -Fc -U postgres -Fc -f /postgres/_to_delete/${DATABASE}.psql ${DATABASE};
      $ scp -3 tadm@db01master.tocco.ch:/postgres/_to_delete/${DATABASE}.psql db1.tocco.cust.vshn.net:
      vshn$ sudo -u potsgres pg_restore -j 4 --role ${DB_USER} --no-owner --no-acl -d ${DB_NAME} ${DATABASE}.psql

      # same again for history DB

* Run CD

* Disable maintenance page

  .. parsed-literal::

      rm /etc/nginx/sites-enabled/100-maintenance-page-\ **${INSTALLATION}**\ .conf
      systemctl reload nginx

* Stop old installation::

      mgrctl stop nice2-${INSTALLATION}

* Adjust DNS entries (where possible)

  See also :doc:`/devops/openshift/dns`

* Test functionality

* Check logs

* Update information in https://www.tocco.ch/tocco

  * Set Server to *VSHN Cloud*
  * Update/remove CD instructions


Post-Move
---------

* Check logs next day

* Check memory next day

* Increase TTL to 3600 seconds again

* Open ticket for required DNS changes

  See also :doc:`/devops/openshift/dns`

* Remove installation at Nine, see :ref:`delete-installation-clean-up-app-server`

* Remove DB dumps::

      tadm@db01master.tocco.ch$ rm /postgres/_to_delete/${DATABASE}.psql
      db1.tocco.cust.vshn.net$ rm ${DATABASE}.psql

      # same again for history DB
