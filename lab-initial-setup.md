
this guide will help you setup a virtualization environment using [gns3](https://www.gns3.com/). most of the instructions given hereafter are links to pages from the official gns3 documentation (the doc contains video and text guides.). this lab was carried on linux; the windows related instructions were not tested.

# Requirements
- working vmware environment. it is possible to use vbox instead of vmware.
- only if you are on linux, working docker engine installed. use instructions from [docker](https://docs.docker.com/engine/install/)


# Prepare the gns3 environment

- start by installing a complete gns3 environment: gns3-gui, gns3-server, the gns3 pre-built virtual machine, etc.
[for linux users](https://docs.gns3.com/docs/getting-started/installation/linux), [for windows users](https://docs.gns3.com/docs/getting-started/installation/windows). All the required and optional components are installed as a single package, but for linux, you'll need to install packages `wireshark` and `gncviewer`.

## Add Cisco devices
- virtual devices are executed by docker, dynamips, or qemu running on a gns3-server, which in turn runs either directly on the host (local server) or on the gns3-vm. if you are using linux, you can use the local server. on windows, you preferably have to use both (some parts of your topology will be run on the gns3-vm). follow [this link](https://docs.gns3.com/docs/getting-started/setup-wizard-local-server#manually-starting-the-setup-wizard) to setup the local server. follow [this link](https://docs.gns3.com/docs/getting-started/setup-wizard-gns3-vm#local-gns3-vm-setup-wizard) to setup the server on the gns3-vm (for Windows users).
 - during this step, you are asked to add cisco devices (run by dynamips), we just need to add the cisco 3640. the image is found [here](http://srijit.com/working-cisco-ios-gns3/). you can use any image. i'am using this one: c3640-ik9o3s-mz.124-25.image.

- before using devices in your topology, you must add them in your gns3 palette inMenu-->Edit-->Preferences. you must beforehand create VMs, add docker images, download cisco images, etc. we'll talk on this in the following.

## Prepare virtual machines and Docker engine
- follow [these instructions](https://docs.gns3.com/docs/emulators/adding-vmware-vms-to-gns3-topologies) to add them to gns3
 - create virtual machines you'll need for your environment. typically, we'll need an ubuntu server (for various services), Windows Server 2012 R2 (for dc), and win7 machines (as workstation). set them to run with minimal RAM requirements, and a single interface.
 - do not forget to install vmware tools inside the virtual machines (drivers to install within the guest os, intended to enable communication between the host and the guest like copy/paste, file sharing, etc.). in VMWare, go to Menu-->VM-->Install VMWare Tools.

- follow [these instructions](https://docs.gns3.com/docs/emulators/docker-support-in-gns3/) in case you want to add docker containers to your gns3 devices palette. on windows, all the docker containers will run in the gns3-vm and the images are downloaded automatically by gns3.


## Adding pfSense appliance
- add the pfSense appliance following [these instructions](https://docs.gns3.com/docs/using-gns3/beginners/install-from-marketplace) and [these pfsense-specific instructions](https://docs-v1.gns3.com/appliances/pfsense.html). pfsense will be used as a network gateway (firewall, vpn server, nat gateway, etc.) and is free.
 - change the requirement RAM to 512Mo instead of 2Go

## Create a connection from host to the gns3 virtual network
- on linux, in order to connect host to gns3 topology: on the host, create a **tap** network interface. this is a virtual interface having an IP address, but instead of being connected to a real layer 2, it is connected to a virtual link. it will be used in order to link the host to the gns3 topology. to create a tap interface in linux, use the following command on you host.
```bash
iface tap0 inet static
        address 10.0.2.253
        netmask 255.255.0.0
        pre-up  /sbin/ip tuntap add dev tap0 mode tap
        post-down /sbin/ip tuntap del dev tap0 mode tap
```
 - alternatively, on windows, follow [these instructions](https://thenetworkberg.com/how-to-add-your-real-computer-to-gns3/)


## Final points
- ask gns3 to show the interface labels on the topology (View-->Show/Hide interface labels)
- 

# Configuration Tasks
- create the topology shown in [this image](https://ibb.co/KVLBv6R).
 - the core-sw is a 3640 cisco layer 3 switch having an additional NM-16ESW network module (in configuration tab of the device)
 - the igw (internet gateway) is a pfSense device
 - the various PCx are based on the gns3 builtin device [VPCS](https://docs.gns3.com/docs/emulators/vpcs). it is an easy to use, lightweight device typically to test connectivity, addressing, etc.
 - wrkstn-1 is a ubuntu server based device

## Setup the core switch interfaces and routing
- create vlans 2 and 3  on the core-sw
- setup interfaces configuration: ip, switchport mode, vlans membership, status
- enable routing
- test intra-vlan and inter-vlan connectivity (proceeds in small steps). you can use the VPCS notes from gns3 as ping sources or targets instead of creating resources consuming virtual machines.

## minimal setup of pfsense
- the idea is make minimal configuration of pfSense using its (vnc-based console) in order to gain access to its beautiful and easy to use web interface. we only have to setup the lan interface for now through this ugly interface.

- manually assign ip address to the `igw.lan` interface in the lan4 subnet (assign 10.0.255.254/24). ensure that the wan interface got an address.

- check connectivity from your host to `igw.lan` using ping, and test the web interface (account is admin/pfsense)
 - ensure you have a route for the entire 10.0./16 block in you host. you can use `ip route add 10.0.0.0/16 core-sw.e0/0` for linux.

- from now on, all the configuration tasks for igw will be done through the web interface (unless otherwise mentioned).

## DHCP
- enable dhcp server on the core-sw to serve dhcp requests (from datacenter, dmz, access subnets). use [this documentation](https://www.cisco.com/c/en/us/td/docs/routers/ir910/software/release/1_1/configuration/guide/ir910scg/swdhcp.pdf) 
 - set ranges (exclude the already assigned addresses)
 
