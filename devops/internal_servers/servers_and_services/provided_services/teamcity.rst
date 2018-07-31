TeamCity
========

TeamCity is our continuous integration server for a lot of stuff: verifying gerrit changes, automatic merges
of the release branches, continuous delivery and more.

Access
------

You can access the TeamCity installation under https://tc.tocco.ch.

Administration
--------------

TeamCity runs as a ``systemd`` service on tc.tocco.ch.

Connect via SSH:

.. parsed-literal::
   $ ssh tadm@tc.tocco.ch

Check if running:

.. parsed-literal::
   $ sudo systemctl status teamcity-server.service

Restart the service:

.. parsed-literal::
   $ sudo systemctl restart teamcity-server.service

.. hint::

    There is also a nightly restart service called ``teamcity-server-restart``.
    Run the following command to check its status:

   .. parsed-literal::
      $ sudo systemctl status teamcity-server-restart.service

Troubleshooting
^^^^^^^^^^^^^^^

Restart docker
""""""""""""""

If restarting the TeamCity service fails, try to restart Docker, as TeamCity runs in a Docker container.
After that, try to start the TeamCity service again.

.. parsed-literal::
   $ sudo systemctl restart docker

Restart the virtual machine
"""""""""""""""""""""""""""

If restarting Docker doesn't help either, try to restart the whole virtual machine.

.. parsed-literal::
   $ sudo reboot
