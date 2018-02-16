DNS
===

.. todo::

    * describe how to add new domains (glue entries, etc.)


Add DNS Entry (Domains Managed by Us)
-------------------------------------

.. hint::

        You probably want to add a :ref:`route <add-route>` first.

#. Go to the `Nine's Cockpit`_.

#. Find the right domain and click on it.

#. Create or update entries

   Let's assume entries for ``tocco.ch``, ``www.tocco.ch`` and ``cockpit.tocco.ch`` are needed, they'd look like this:

   .. code-block:: ini

       ; domain itself
       tocco.ch          3600 IN A      5.102.151.2
       tocco.ch          3600 IN A      5.102.151.3

       ; subdomains
       www.tocco.ch      3600 IN CNAME  ha-proxy.tocco.ch
       cockpit.tocco.ch  3600 IN CNAME  ha-proxy.tocco.ch

   Note that for domains themselves **two** ``A`` entries are needed. For subdomains only one ``CNAME``.

   In Nine's web interface, add domains like this …

       .. figure:: dns_static/nine_a_record.png
           :scale: 60%

           add record\ **s** for domain (IPs ``5.102.151.2`` and ``5.102.151.3``)


   … and subdomains like this:

       .. figure:: dns_static/nine_cname_record.png
           :scale: 60%

           add record for subdomain (target ``ha-proxy.tocco.ch``)

#. Remove superfluous entries

    Remove all other ``A`` and ``CNAME`` entries for the domains/subdomains. So, only the ones you created/adjusted
    remain. **Don't touch any other entries though.**

.. _Nine's Cockpit: https://cockpit.nine.ch/en/dns/domains


Add DNS Entry (Domains Managed by a Third Party)
------------------------------------------------

.. hint::

        You probably want to add a :ref:`route <add-route>` first.

The entity that controls the domain needs to add the following entries.

.. code::

    www.example.com.    IN CNAME ha-proxy.tocco.ch.
    example.com.        IN A     5.102.151.2
    example.com.        IN A     5.102.151.3

All subdomains, e.g. *intranet.example.com*, *www.zrh.example.com*, *www.example.com* should use a ``CNAME`` entry.
The domain itself, however, needs to have two ``A`` entries because DNS doesn't allow ``CNAME``\ s on them,
unfortunately.
