DNS
===

.. todo::

    * describe how to add new domains (glue entries, etc.)


.. _what-dns-records-are-needed:

What DNS Records are Needed?
----------------------------

Let's assume a customer called **bmx** owns the domains bmx.ch, zero.net and wants ``bmx.ch``, ``www.bmx.ch``,
 ``extranet.bmx.ch`` and ``extranet.zero.net`` to be served by Tocco:

.. code-block:: ini

    ;;; ${CUSTOMER}.tocco.ch ;;;
    ; All customers should have an *.tocco.ch entry
    bmx.tocco.ch.      3600 IN CNAME  ha-proxy.tocco.ch

    ;;; domain itself ;;;
    ; CNAME not allowed on domains. Thus, IPs must be used.
    bmx.ch.            3600 IN A      5.102.151.2
    bmx.ch.            3600 IN A      5.102.151.3

    ;;; subdomains ;;;
    ; Point to domain, bmx.ch in this case, when possible.
    www.bmx.ch.        3600 IN CNAME  bmx.ch.
    extranet.bmx.ch.   3600 IN CNAME  bmx.ch.
    ; Point to ${CUSOTMER}.tocco.ch if domain doesn't point to Tocco (no zero.net entry).
    extranet.zero.net. 3600 IN CNAME  bmx.tocco.ch.
                                                     

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

.. _dns-managed-by-us:

Add DNS Record for a Domain managed by Us
-----------------------------------------

.. hint::

        You probably want to add a :ref:`route <ansible-add-route>` first.

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

           add record for subdomain (target ``ha-proxy.tocco.ch`` or ``${DOMAIN}``)

       If you add a subdomain and the domain itself is served by Tocco, use ``${DOMAIN}``
       as target. For instance, if site.net is served by us and you add www.site.net set
       *Target* to ``site.net``. For ``${CUSTOMER}.tocco.ch``, and other subdomains where
       the domain points somewhere else, set *Target* to ``ha-proxy.tocco.ch``.

#. Remove superfluous entries

    Remove all other ``A`` and ``CNAME`` entries for the domains/subdomains. So, only the ones you created/adjusted
    remain. **Don't touch any other entries though.**

.. _Nine's Cockpit: https://cockpit.nine.ch/en/dns/domains


Add DNS Record for Domains Managed by a Third Party
---------------------------------------------------

.. hint::

        You probably want to add a :ref:`route <ansible-add-route>` first.

Since we won't have any control over the DNS server, you'll have to communicate the customer the information in
:ref:`what-dns-records-are-needed`, so they can ensure the entries are created.

.. _verify-dns-records:

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
