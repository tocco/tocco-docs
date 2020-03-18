###
DNS
###

DNS Changes
===========

This is what you do if a customer requests a DNS change:

#. Verify that we can make the changes:

   See :ref:`dns-who-updates-record`

#. Execute the changes keeping this in mind:

  a) If the customer requests you add or adjust an SPF entry. Adjust the SPF
     record as requested but make sure *include:spf.tocco.ch* is part of
     the record. See :ref:`create-adjust-spf-record` for details.
  b) If the customer wants to setup a new domain served by Tocco, follow
     the instruction in `Add new (Sub-)Domain in CMS (VSHN)`_ or `Add new
     (Sub-)Domain in CMS (Nine)`_ respectively.
  c) For any other request, execute the change on https://cockpit.nine.ch
     as requested.


Add new (Sub-)Domain in CMS (VSHN)
==================================

#. Create the required DNS entries:

   see :doc:`/devops/openshift/dns`

#. Add route and issue TLS certificate

   **At this point the DNS entry must exist.**

   See :doc:`/devops/openshift/routes`

#. Add URL to domain

   #. Open *Web* → *Redaktionssystem*
   #. Double click domain to edit (tree on left)

      *URL* is the main domain and *Aliases* are additional aliases. All
      aliases redirect to the main *URL*.

      Add an http\:// and https\:// URL for all domains. For instance, to
      make the page reachable at *example.net* and *www.example.net* add
      this URLs::

          http://example.net
          https://example.net
          http://www.example.net
          https://www.example.net

      If the domain should be use as main domain, add it as URL. **Always
      use a https:// URL as main domain**.

#. Test if URL is reachable

   In case you see a certificate error checkout the *Troubleshooting* section
   in :doc:`/devops/openshift/routes`.

   Verify the correctness of the DNS entries. For instance by using `this tool
   <https://dnslookup.org/tocco.ch/A/#delegation>`__. Ensure only one *A* or
   *CNAME* entry exists. Also, ensure no *AAAA* entry exists.


Add new (Sub-)Domain in CMS (Nine)
==================================


#. Create the required DNS entries:

   First, find the IP / CNAME to use. For this lookup what's used for
   ${installation}.tocco.ch::

        $ dig agogis.tocco.ch
        ;; ANSWER SECTION:
        example.tocco.ch.     3600    IN CNAME  container04.tocco.ch.
        container04.nocco.ch.	3600    IN A      94.230.213.26

   (`online verison <https://dnslookup.org/customer.tocco.ch/A/#delegation>`__)

   Now use the IP (*94.230.213.26*) for the domain and the CNAME (*container04.tocco.ch*)
   for subdomains. Example::

       ; domain
       example.net.          IN A    94.230.213.26

       ; subdomains
       www.example.net.      IN A    container04.tocco.ch.
       intranet.example.net  IN A    container04.tocco.ch.

   Follow the :doc:`instructions for VSHN </devops/openshift/dns>` but
   use these IP and CNAME.

#. Add hostname(s) to Nginx config:

   First, find the right config file::

       $ ssh agogis.tocco.ch

       # find the right config file
       $ grep 'server_name .*agogis.tocco.ch' /etc/nginx/sites-enabled/*

   Then add the new hostnames as *server_name*. Like this::

       server_name example.tocco.ch example.net www.example.net intranet.example.net;

   Finally, reload nginx::

       sudo nginx -s reload

#. Next a TLS certificate needs to be issued.

   **At this point the DNS entry must exist.**

   See section *Extending an Existing Certificate* in `ACME / Certbot on App Servers`_.

#. Add URL to domain

   See *Add URL to domain* for VSHN above.

#. Test if URL is reachable

   Verify the correctness of the DNS entries for instance by using `this tool
   <https://dnslookup.org/tocco.ch/A/#delegation>`__. Ensure either exactly
   two *A* records exist or exactly one CNAME record. Also, no *AAAA* record
   must exist.


.. _dns-who-updates-record:

Who has to Update the DNS Record?
=================================

Step one is to find out if we manage DNS for the domain ourselves::

    $ dig -t ns +short ${DOMAIN}
    ns1.tocco.ch.
    ns2.tocco.ch.

Or use an `online tool <https://dnslookup.org/tocco.ch/NS/#delegation>`__

**If the NS (name servers) are ns1.tocco.ch and ns2.tocco.ch, the
domain is managed by us** and you can login on https://cockpit.nine.ch
to manage the domains. Otherwise, the domain is managed by the customer
or a third pary. In such a case inform the customer about needed changes.


.. _ACME / Certbot on App Servers: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=docs/services/app_server_acme.rst;hb=HEAD
