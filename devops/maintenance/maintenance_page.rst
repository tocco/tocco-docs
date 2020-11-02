Maintenance Page
################

Repository
==========

The script is hosted as a gitlab repository and be available under https://gitlab.com/toccoag/maintenance-page/

The customer themes are hosted as a separated gitlab repository https://gitlab.com/toccoag/maintenance-page-customer .
The default branch is the fallback if no customer branch exists.

Usage
=====

1. Login ``oc login``
2. Go to the project where the maintenance page should be deployed (e.g. ``oc project toco-nice-test212``)
3. Enable maintenance page ``mntnc start [-c tocco] [-t MyTitle] [-b "Downtime until 12am."] [-r 10]``

The following optional arguments are supported:

* ``-c``/``--customer``: branch picked of repository where the customer theme is defined. If no argument is passed or the branch does not exist the default branch is taken.
* ``-t``/``--title``: title of the page (default: ``Wartungsarbeiten``)
* ``-b``/``--body``: body of the page (html elements are supported e.g. ``-b "<b>Currently offline</b>``; default: ``(empty string)``)
* ``-r``/``--replicas``: number of replicas deployed (default: ``1``)

4. Disable/enable bypass as needed via https://test212.tocco.ch/_maintenance/bypass/
5. Disable maintenance page ``mntnc stop``

Theme
=====

Here is an example on how the theme should look like:

.. code-block:: HTML

    <!DOCTYPE html>
    <html lang="de">
       <head>
        <meta charset="utf-8">
        <meta name="robots" content="noindex, nofollow">
        <title>{{{TITLE}}}</title>
       </head>
       <body>
          <h1>{{{TITLE}}}</h1>
          <div>The system is currently in maintenance mode. {{{BODY}}}</div>
       </body>
    </html>

The HTML code should contain two placeholders ``{{{TITLE}}}`` and ``{{{BODY}}}`` which are set over the ``mntnc`` command.
Note that the body can be an empty string.
