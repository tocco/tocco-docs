Nginx Configuration
===================

This document describes the available configuration for the `Docker image used on OpenShift <https://github.com/tocco/openshift-nginx>`__.

Custom HTTP Headers
-------------------

Custom HTTP headers can be configured using ``NGINX_HEADER_*`` env. variables.

Set header for responses with **2XX status codes**::

    NGINX_HEADER_Content__Security__Policy="default-src 'none'; block-all-mixed-content; connect-src 'self' example.net"

This creates a header called ``Content-Security-Policy`` with the value ``default-src 'none'; block-all-mixed-content; connect-src 'self' example.net``.

Set header on **all** responses regardless of status code::

    oc set env -c nginx dc/nice NGINX_ALWAYS_HEADER_Strict__Transport__Security='max-age=62208000'

This creates a header called ``Strict-Transport-Security`` with the value ``max-age=62208000``.

.. hint::

    OpenShift does not currently allow hyphens (``-``) to appear in keys of environment variables,
    you need to use double underscores (``__``) instead. (See examples above.)


IP Whitelisting
---------------

Nginx can be configured to only allow access from certain IPs using the ``IP_WHITELIST``
env. variable. Request from any other IP address is blocked by replying with *403 - Forbidden*.

Example::

    oc set env -c nice dc/nice IP_WHITELIST='81.55.33.3 40.22.0.0/16 1.5.5.5-1.5.5.7'

This configuration only allows access from the IP address ``81.55.33.3``, the CIDR ``40.22.0.0/16`` and
IPs in the range ``1.5.5.5`` to ``1.5.5.7`` inclusive.
