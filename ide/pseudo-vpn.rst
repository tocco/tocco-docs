Pseudo-VPN using SOCKS5 over SSH
################################

This page documents how SSH can be used as a pseudo-VPN. First, a
SSH tunnel needs to be opened with a local SOCKS5 endpoint. Once
that's done, any application supporting SOCKS5 (e.g. Firefox) can
be configured to tunnel traffic through the tunnel.

.. hint::

    For admins:

    In order to grant tunnel access only, grant *@remote* access
    to the user as described in :ref:`ssh-server-access-ansible`.


Setup SOCKS5 Tunnel on Linux
============================

#. If you haven't setup up ssh yet, set it up according to :ref:`set-up-ssh`.

#. Open tunnel::

      $ ssh -D 3333 -N tocco-proxy@git.tocco.ch


Setup SOCKS5 Tunnel on Windows
==============================

#. `Download and install PuTTY`_

#. Open Putty Key Generator and generate a key:

   .. figure:: resources/putty_0_generate_key.png

        Putty Key Generator

   1. Select type *Ed25519*.
   2. Generate a key.
   3. Save your private key. Remember the location.
   4. Copy the public key and forward it to operations to
      allow them to grant to you access.

#. Open Putty and configure it as follows:

   .. figure:: resources/putty_1_load_key.png

         Connection → SSH → Auth

   Use the key you stored in the previous step.

   .. figure:: resources/putty_2_no_command.png

        Connection → SSH

   Check *Don't start a shell or command at all*.

   .. figure:: resources/putty_3_socks5.png

        Connection → SSH → Tunnels

   .. figure:: resources/putty_4_username.png

        Connection → Data

   Use *tocco-proxy* as username.

   .. figure:: resources/putty_5_save_session.png

        Session

   1. Set host name *git.tocco.ch* (or alternively *backup02.tocco.ch*).
   2. Set port 32711.
   3. Set a name for the session.
   4. Save the session.

#. Open tunnel:


   .. figure:: resources/putty_6_open_tunnel.png

        Session

   Once the session has been saved, double click on the name to connect
   and open a pseudo-VPN tunnel.

   To connect in the future, open Putty again and repeat this step.


Use SOCKS5 Tunnel in Firefox
============================

Once the tunnel is open, you can configure a SOCKS5 proxy in Firefox
to use it as pseudo-VPN.

#. Open *Settings* in Firefox
#. Search and open *Network Settings*

   .. figure:: resources/firefox_0_network_settings.png

#. Set a proxy

   .. figure:: resources/firefox_1_proxy_settings.png


.. _download and install PuTTY: https://www.chiark.greenend.org.uk/~sgtatham/putty/
