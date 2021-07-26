Styling
==============
If tocco apps, for example tocco-login or entity-browser, are integrated in 3rd party applications or content management systems, there 
is the need to customize the apps in terms of the visual appearance to match the rest of the application.

Custom Theme
-------------
Every app supports the input property *customTheme*, which is an object with a defined structure.
Inside the tocco-theme package there is a default-theme defined. Whenever no *customTheme* is provided to an app, the default theme will be used
as a fallback. If a *customTheme* is provided with only a few properties defined (e.g. only the colors), the *customTheme* will be merged with the default theme.

The `Default Theme`_ can be found here.

.. _Default Theme: https://github.com/tocco/tocco-client/blob/master/packages/tocco-theme/src/ToccoTheme/defaultTheme.js


This is a valid example of a *customTheme* only changing the primary color and the used font:

.. code-block:: json           

  {
      "colors": {
          "primary": "#36ff00"
      },
      "fontFamily": {
          "regular": "'Helvetica Neue', Helvetica, Arial, sans-serif"
      }
  }



Nice2 Integration
------------------
Apps used in Nice2 are only using the default theme at the moment. But apps used as widgets in the cms support a standardized way of adding a customTheme.

The following file should define a global variable named ``customerTheme`` containing a valid *customTheme* (or parts of it): 
``customer/{CUSTOMER}/share/cms/web/js/tocco-client-theme.js``

In the `page_header.ftl` this file is loaded and the `reactRegistry.js` will check, if the global variable is defined and passes it to the tocco app if yes.

Of course this only gives control over a handful of parameters. If more customizing is wanted by the customer, there is still the way to add css on top of everything. 
But such styling will never be supported and might break with an update of a client packages.



