Edge Remote Desktop
===================

Installation
------------

#. Microsoft provides dedicated virtual machines for testing edge and ie11. Download it `here <https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/>`_ .

   .. note:
     
      Yes Microsoft doesn't provide a simple .iso file. You can just download a prebuilt image for different VM platforms. That's why we have to convert the image.
      Suggested is to download the Virtualbox (.ova) image. It actually doesn't matter what image you download but he following steps fit to a Virtualbox image.

 
#. After downloading we have to convert into the KVM format (qcow2). 

   .. code::

      tar -xf MSEdge-Win10.ova
     
      ll (ls -alh) ->
      -rw-------  1 nikr nikr 7.0K Oct 23 19:37 'MSEdge - Win10.ovf'
      -rw-------  1 nikr nikr 5.0G Oct 23 19:47 'MSEdge - Win10-disk001.vmdk'
      -rw-r--r--  1 nikr nikr 5.0G Oct 23 10:47  MSEdge-Win10.ova

      time qemu-img convert -f vmdk 'MSEdge - Win10-disk001.vmdk' -O qcow2 MSEdge-Win10-disk001.qcow2

      ll ->
      -rw-r--r--  1 nikr nikr  11G Dec 19 11:12  vm-102-disk-1.qcow2


#. Now we have our KVM image ready. We just have to upload it. We can simply do that via the Proxmox web interface.

   .. figure:: edge/upload_image_to_proxmox.png


#. Create a new Virtual Machine exactly like shown in the following picture sequence.
   You can choose any number you like for the VM ID.

   .. figure:: edge/create_vm1.png

   .. figure:: edge/create_vm2.png

   .. figure:: edge/create_vm3.png

   .. figure:: edge/create_vm4.png

   .. figure:: edge/create_vm5.png

   .. figure:: edge/create_vm6.png

   .. note:: 
      
      the VitrIO network device won't work for windows because it doesn't have installed the dirvers. That's why we to install them manually.


#. At this moment you should have successfully create a new VM. It should appear on the left side by the other VM's
   For the next step we need to command line again. We have to replace to mock image with the windows edge image.

   .. code::

      ssh tadm@host03b.tocco.ch

      VMID=*the nuber you have choosen for you vm*

      cd /home/tadm/backup/storage/images/${VMID}

      rm vm-102-disk-1.qcow2

      exit

      scp vm-102-disk-1.qcow2 tadm@host03b.tocco.ch:/home/tadm/backup/storage/images/${VMID}

      ssh tadm@host03b.tocco.ch

      sudo qm rescan -vmid ${VMID}


#. Download divers for the VitrIO network device.

   The VM is now ready to boot, but we need the drivers for the network device else we won't have any connection to the network.
   Fedora provides the driver for Windows. It can be found `on the fedora project website <https://fedoraproject.org/wiki/Windows_Virtio_Drivers#Direct_download>`_.

   We have to upload the driver the same way we did it with the image file, see step 3.
   Before we can start the VM we need to add the drivers to a virtual drive, so that you can access it inside the VM.

   .. figure:: edge//upload_vitrio_driver.png


#. The VM should now be ready to boot. 
   For that, click on the VM on the left side and then on the start button in the right top corner.

   .. figure:: edge/start_vm.png

   .. hint::

      When the vm starts a Windows logo should appear. After booting you should be logged in automatically. 
      If not (what might be possible it is microsoft), the default password for Microsoft VM's is **Passw0rd!**.


#. Install the VirtIO drivers

      * open the **Device Manager**

      * open the tab **Network adapters** 

      * left click on **Red Hat VirtIO Ethernet Adapter**

      * **Properties**

      * open tab Driver

      * click the button **Update Driver**

      * choose **Browse my computer for updated driver software**

      * Select the drive with the driver in it. It will scan the drive an automatically recognize the driver and install it.

      * After installing the driver you probably have to reboot the VM.


#. IP configuration
   When we finally got a connection to the network we need to configure the IP address. In the end edge.tocco.ch should point to the VM.

   Usually edge.tocco.ch points to 10.27.1.33. But this might have changed when you are reading this. So check the DNS entry before. 
   You can do that in the Linux shell: 
   
   .. code::
 
      dig edge.tocco.ch -> ;; ANSWER SECTION:
      edge.tocco.ch.          43200   IN      A       10.27.1.33


   So you see edge.tocco.ch points to 10.27.1.33

      * left click the **screen symbol** at the **bottom right corner**

      * right click the option **Open Network & Internet settings**

      * right click **Change adapter options**

      * left click on the interface: **Ethernet 3**

      * right lick on **properties**

        .. figure:: edge/ip_configuration1.png

      * right click the check box **Internet Protocol Version 4 (TCP/IPv4)** and click on **Properties**

      * Fill in the form as follows:

        .. figure:: edge/ip_configuration2.png


#. Finally we can activate Microsoft Remote Desktop

      * Type **Settings** in to the **search form** at the **bottom left corner**.

      * click on the **Settings** Symbol.

      * click on **System**

      * Click in the **list on the left side** on **Remote Desktop**

      * Toggle the checkbox to **On** to enable Microsoft Remote Desktop

      * If you go through the form you will see this sentence:
        **Use this PC name to connect from your remote device**

   Not sure if it has really an effect but to be save change the PC name to edge.


#. Now we got the VM connected to the Network and Microsoft Remote Desktop running.
   Unfortunately this doesn't mean that it works.

   If you do a network scan you will recognize that there is no visible Host with the given IP. Even ping doesn't work.
   To test that you can use the following commands:

   .. code::

      nmap -sP 10.27.1.*

      ping edge.tocco.ch


   The solution for this Problem is simple. Just deactivate all firewall's.

      * Type **firewall** in to the search field at the **left bottom corner**.

      * **Windows Defender Firewall** will appear as a search result. Click on that

      * Click on the Tab **Turn Windows Defender Firewall on or off** in the list at left side.

      * Disable the firewall **Public and Private** by clicking on the **Turn off** radio button's.

      * click on the **OK** button to save the setttings. 

      * You probably have to restart the VM again.

#. Now we can try open a remote desktop.
   On Linux **rdesktop** is recommended 

   .. code::

      rdesktop edge ->

      ERROR: CredSSP: Initialize failed, do you have correct kerberos tgt initialized ?
      Failed to connect, CredSSP required by server.

  .. attention:: 
     
     It is very likely that thiy error will appear now. to get rid of that message hjust disable CredSSP. To avoid that you have to search 1 hour for the checkbox below is shown how to find it.


#. Disable CredSSP
   
      * Type **System** in the search field at the **left buttom corner**. Click on **System** as it appears.

      * On the **left list** click on the tab **Remote Settings** 

      * A new windows opens. At the bottom you find an enabled check box it is labeled as follows:
        **Allow connections onyl from computer running Remote Desktop with Network Level Authentification**
        Disable the checkbox.

   Now you should be able to open a Remote Desktop Session.


# Miscellaneous

    * Change the User name to tocco.

    * Change the password to tocco standard password.

    * Make a copy/backup and never ever touch that thing again!!
