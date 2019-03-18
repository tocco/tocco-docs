Cross-Origin Resource Sharing (CORS)
====================================

By default, the REST resources cannot be accessed from another domain outside the domain from which the REST API is
served (forbidden by the `same-origin security policy`_).

`Cross-Origin Resource Sharing (CORS)`_ defines a way to allow those cross-origin requests which are forbidden by default.

This works by setting some HTTP response headers on our REST responses. The easiest way to achieve this is by adding
them to the ``nice2.web.core.ResponseHeaders`` configuration point (while restricting the headers to the URLs which start
with `nice2/rest/`).

.. _same-origin security policy: https://en.wikipedia.org/wiki/Same-origin_policy
.. _Cross-Origin Resource Sharing (CORS): https://en.wikipedia.org/wiki/Cross-origin_resource_sharing

Headers
-------

The HTTP headers that relate to CORS are

Request headers
^^^^^^^^^^^^^^^

- Origin
- Access-Control-Request-Method
- Access-Control-Request-Headers

Response headers
^^^^^^^^^^^^^^^^

- Access-Control-Allow-Origin
- Access-Control-Allow-Credentials
- Access-Control-Expose-Headers
- Access-Control-Max-Age
- Access-Control-Allow-Methods
- Access-Control-Allow-Headers

Example configuration
---------------------

The following simple example allows all kinds of REST requests from the domain `www.someotherdomain.ch`, while
cross-origin requests from all other domains are still forbidden.

To allow only a subset of the HTTP methods or request headers, simply adjust the corresponding ``<header>``
contribution according to your needs.

Put the following XML code in the `hivemodule.xml` of the customer you want to enable cross-origin requests for
(file location: `customer/${CUSTOMER}/module/module/descriptor/hivemodule.xml`).

.. code-block:: xml

  <contribution configuration-id="nice2.web.core.ResponseHeaders">
    <header name="Access-Control-Allow-Origin"
            value-supplier="service:nice2.web.core.AccessControlAllowOriginValueSupplier"
            url-pattern="/nice2/rest/.*"/>
    <header name="Vary"
            value="Origin"
            url-pattern="/nice2/rest/.*"/>
    <header name="Access-Control-Allow-Credentials"
            value="true"
            url-pattern="/nice2/rest/.*"/>
    <header name="Access-Control-Allow-Headers"
            value="Authorization,Content-Type,X-Business-Unit"
            url-pattern="/nice2/rest/.*"/>
    <header name="Access-Control-Expose-Headers"
            value="Location"
            url-pattern="/nice2/rest/.*"/>
    <header name="Access-Control-Allow-Methods"
            value="GET,POST,PUT,DELETE,OPTIONS,PATCH"
            url-pattern="/nice2/rest/.*"/>
  </contribution>

Additionally, add the following application property:

.. code-block:: Properties

    nice2.web.allowedRequestOrigins=https://www.someotherdomain.ch

.. note::

   The ``nice2.web.core.ResponseHeaders`` configuration point can be used for all HTTP response headers you want to set
   on certain HTTP responses provided by the Nice2 application. Setting the CORS headers is just one possible use case.

AccessControlAllowOriginValueSupplier
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You might have noticed that the ``<header>`` contribution for the header ``Access-Control-Allow-Origin`` uses the
attribute ``value-supplier``, while all other contribution use ``value`` with a hardcoded string value.

If you intend to enable CORS only for one domain, you'd be perfectly fine by hardcoding that single domain in the
`hivemodule.xml`. The header contribution for ``Access-Control-Allow-Origin`` could be replaced with the following
contribution in this case:

.. code-block:: xml

    <header name="Access-Control-Allow-Origin"
            value="https://www.someotherdomain.ch"
            url-pattern="/nice2/rest/.*"/>

Also, you could remove the ``nice2.web.allowedRequestOrigins`` application property.

However, as soon as cross-origin requests should be allowed for more than one domain or for all domains, we need a
more dynamic response header value. In this case, the ``Origin`` request header has to be set as value for the
``Access-Control-Allow-Origin`` response header. And that's exactly what the ``AccessControlAllowOriginValueSupplier``
is here for. It sets the value of the ``Origin`` request header as value of the response header, if the origin
is allowed according to the ``nice2.web.allowedRequestOrigins`` application property.

Enable CORS for multiple domains
''''''''''''''''''''''''''''''''

Let's say, we'd like to enable CORS for the origins `https://www.someotherdomain.com` and `https://www.example.com`.
In this case, we'd have to use the ``AccessControlAllowOriginValueSupplier`` and set the application property as follows:

.. code-block:: Properties

    nice2.web.allowedRequestOrigins=https://www.someotherdomain.ch,https://www.example.com

.. note::

   The number of allowed request origins is not limited. You can add as many origins as you want (separated by comma).

Enable CORS for all domains
'''''''''''''''''''''''''''

If you want to enable CORS for **all** domains, use the ``AccessControlAllowOriginValueSupplier`` and **don't** set
the ``nice2.web.allowedRequestOrigins`` application property (don't set it at all or leave it empty).
