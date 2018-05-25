DNS
===

.. todo::

    * describe how to add new domains (glue entries, etc.)


.. _what-dns-records-are-needed:

What DNS Records are Needed?
----------------------------

Let's assume entries for ``tocco.ch``, ``www.tocco.ch`` and ``cockpit.tocco.ch`` are needed, they'd look like this:

.. code-block:: ini

    ; domain itself
    tocco.ch          3600 IN A      5.102.151.2
    tocco.ch          3600 IN A      5.102.151.3

    ; subdomains
    www.tocco.ch      3600 IN CNAME  ha-proxy.tocco.ch
    cockpit.tocco.ch  3600 IN CNAME  ha-proxy.tocco.ch

Note that for domains themselves **two** ``A`` records are needed and for subdomains only one ``CNAME``.


Find out if Domain is Managed by Us
-----------------------------------

Obtain a list of DNS server for the domain:

    .. parsed-literal::

        dig -t ns **${DOMAIN}}**

    If the result contains the name servers ``ns1.tocco.ch`` and/or ``ns2.tocco.ch``, the domain is hosted by us.
    Otherwise, it is not.

    .. hint::

        **${DOMAIN}** is the domain part only, e.g. tocco.ch and not :strike:`www.tocco.ch` or
        :strike:`cockpit.tocco.ch`.


Add DNS Record for a Domain managed by Us
-----------------------------------------

.. hint::

        You probably want to add a :ref:`route <add-route>` first.

#. Go to `Nine's Cockpit`_.

#. Find the right domain and click on it.

#. Create or update entries

   In Nine's web interface, add **domains** like this …

       .. figure:: dns_static/nine_a_record.png
           :scale: 60%

           add record\ **s** for domain (IPs ``5.102.151.2`` and ``5.102.151.3``)


   … and **subdomains** like this:

       .. figure:: dns_static/nine_cname_record.png
           :scale: 60%

           add record for subdomain (target ``ha-proxy.tocco.ch``)

#. Remove superfluous entries

    Remove all other ``A`` and ``CNAME`` entries for the domains/subdomains. So, only the ones you created/adjusted
    remain. **Don't touch any other entries though.**

.. _Nine's Cockpit: https://cockpit.nine.ch/en/dns/domains


Add DNS Record for Domains Managed by a Third Party
---------------------------------------------------

.. hint::

        You probably want to add a :ref:`route <add-route>` first.

Since we won't have any control over the DNS server, you'll have to communicate the customer the information in
:ref:`what-dns-records-are-needed`, so they can ensure the entries are created.


Verify DNS Records
------------------

Get **A** records for host:

    .. parsed-literal::

        dig **${HOSTNAME}**

Verify output:

    The ``ANSWER SECTION`` of the output must contain the following **A** entries::

        ... IN A 5.102.151.2
        ... IN A 5.102.151.3

    The output may also contain ``CNAME`` entries. However, it **must not** contain any other **A** entries. If it does,
    they must be removed.
