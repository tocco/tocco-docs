DNS
===

.. todo::

    * describe how to add new domains (glue entries, etc.)


Add DNS Entry (Domains Managed by Us)
-------------------------------------

.. hint::

        You probably want to add a :ref:`route <add-route>` first.

1. Go to the `Nine's Cockpit`_.

2. Find the right domain and click on it.

3. Find the "create" button.

    .. figure:: dns_static/create_button.png
        :scale: 60%

        create button

4a. For the domain itself you need **two** ``A`` entries …

    .. figure:: dns_static/nine_a_record.png
        :scale: 60%

        add record\ **s** for domain (IPs ``5.102.151.2`` and ``5.102.151.3``)

4b. … and for subdomains a ``CNAME`` entry

    .. figure:: dns_static/nine_cname_record.png
        :scale: 60%

        add record for subdomain (target ``ha-proxy.tocco.ch``)

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
