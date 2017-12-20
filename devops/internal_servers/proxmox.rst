Proxmox
=======

PROXMOX is an open source virtualization tool. It allows you to run several virtual machines on one single server.
We use Proxmox to provide several internal services like: Postgres or Teamcity.

Setup
-----

Here is explaind how to set up PROXMOX if one of the hosts fails.

 #. Grab the key for the Tocco server room and a pair of ear plugs. (it might be very loud)

 #. Prepare yourself a stick with the latest Debian image.

 #. Setup Debian on the server but consider some points.

    * set user root with the right root password.
    * set the right domain name e.g. host03a.tocco.ch
    * set the right ip address for the used domain name

 #. follow the `installation manual <https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_Stretch#Install_Proxmox_VE>`_ for Proxmox.
