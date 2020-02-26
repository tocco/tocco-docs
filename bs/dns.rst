###
DNS
###

Add new (Sub-)Domain
====================

TODO (dns entries, TLS certificate, CMS configuration)


Requests to Change DNS Entries
==============================

TODO


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
