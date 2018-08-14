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

If you cannot access the virtual machine at all, try restarting/resetting the machine via the Proxmox web interface:

1. Open Proxmox web interface in the browser: https://host03a.tocco.ch
2. Log in with `root`
3. Find the machine **115 (tcserver01)** in the navigation menu on the left
4. Press the "Start" button on the right if the machine is not running or select "Reset" from the "Shutdown" menu

Restart an agent
""""""""""""""""

If one of the agents is not available, check via Proxmox web interface if the virtual machine is running and restart
it if necessary.

1. Look up the host machine here: :doc:`index`
2. Open Proxmox web interface in the browser: https://**${HOST}**.tocco.ch (e.g. https://host03c.tocco.ch for TC-Agent-4)
3. Log in with `root`
4. Find the agent machine in the navigation menu on the left
5. Press the "Start" button on the right if the machine is not running or select "Reset" from the "Shutdown" menu
