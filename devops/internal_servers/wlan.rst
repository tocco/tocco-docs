Wlan Netwotk
============

The Network
-----------

Tocco has an internal Wlan's consisting of three Unifi ``AP's`` [#f1]_

  ================== ================== ================== ==================
   Type               Location           Domain             IP
  ================== ================== ================== ==================
   UniFi AP-AC-LR     2nd floor YOGA     unifi03.tocco.ch   10.27.1.54
   UniFi AP-Pro       3rd floor          unifi02.tocco.ch   10.27.1.56
   UniFi AP-AC v2     2nd floor          unifi01.tocco.ch   10.27.1.55
  ================== ================== ================== ==================

Access The Control Software
---------------------------

The network and the ``AP's`` can be controlled via the **Unifi Controller Software**. It is installed on the edge remote desktop vm on host03b. 
You can install it on your local machine but it is not recommended. You probably have to reconfigure all ``AP's``. That means that all wireless networks will be reset.
Only do it in case of a broken/unavailable edge vm or an urgent "wlan emergency". If so, follow the `installation manual <https://dl.ubnt.com/guides/UniFi/UniFi_Controller_UG.pdf>`_ under Chapter 1: System Setup.

#. Once you have successfully logged on to the edge vm start the UniFi Controller. There is a desktop icon for it.

#. Wait until the Controller is ready and then click the button named **Launch a Browser to manage the Network**.

#. A Browser will open with a login form. The credentials are **tocco/standard password**

#. The UniFi Control Panel will open.


Unifi Controll Panel
--------------------

The Control Panel has a very intuitive design but there are two basic interfaces which we need to configure/maintenance the wlan's.

#. The Panel for Devices. Here you can manage all the ``AP's`` (update, restart).

   .. figure:: wlan/unifi_device_panel.png

#. The Panel for wlan's. You can find it in the settings tooltip. Just click on the button named **Wireless Networks**.
   Then The panel opens and you see all the available wlan's. In case of a reset of the AP's you have to configure the wlan's like in the second picture.

   .. figure:: wlan/unifi_network_setting_panel.png

   .. figure:: wlan/unifi_wlan_setting_panel.png


.. [#f1] Abbreviation for Access Point.

