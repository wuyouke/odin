Odin
====

SDN-based solution for programmatic orchestration of enterprise WiFi networks.

I would like to state clear that this project is not my own idea, the repository found here is a fork of the original Odin framework (https://github.com/lalithsuresh/odin) by Lalith Suresh. New features have been added:

* deauthentication management
* odin-agents virtualization
* script for automatic roll-out
* fixed minor bugs in click (i.e. padding)

References
----------

* http://conferences.sigcomm.org/sigcomm/2012/paper/hotsdn/p115.pdf
* https://www.usenix.org/system/files/conference/atc14/atc14-paper-schulz_zander.pdf

The system is described in the following Masters' thesis: http://lalithsuresh.files.wordpress.com/2011/04/lalith-thesis.pdf

Requirements
------------

* Click modular router
* Java Development Kit
* Docker
* xDPd 
* ROFL 
* hostapd
* Wireless card based on atheros driver ath9k_htc, ath9k or ath5k

xDPd and rofl-core are assumed to be built already, if you don't know how to do that you can get the instructions at:

* https://github.com/bisdn/xdpd
* https://github.com/bisdn/rofl-core

Building instructions
---------------------

If you cloned Odin from the git repository, pull the individual submodules (odin-agent, odin-master, odin-click and odin-utilities):

```
  $: git clone  https://github.com/fgg89/odin.git
  $: cd odin
  $: git submodule init
  $: git submodule update
```

Building click modular router
-----------------------------

The odin-agent is a module inside click (odinagent.cc and odinagent.hh). For easier development we can create a link for both files inside click/elements/local:

```
  $: cd odin-agent/src
  $: ln odinagent* ../../odin-click/elements/local/
```

Then we can build click with the following options:

```
  $: cd odin-click
  $: ./configure --enable-local --enable-wifi --disable-linuxmodule
  $: make -j4
```

If you want to cross-compile click for running it under OpenWRT you may want to check the following links:

* http://wiki.openwrt.org/doc/howto/buildroot.exigence
* http://wiki.openwrt.org/doc/devel/crosscompile

Building the odin-master
------------------------

```
  $: cd odin-master
  $: ant
```

The floodlight.jar is located under /target. 

Patching the driver module
--------------------------

For further instructions please visit https://github.com/fgg89/odin-utilities/tree/master/ath9k_htc

Running Odin
------------

The master is to be run on a central server that has IP reachability to all APs in the system. The master expects the following configuration parameter to be set in the floodlight configuration file (src/main/resources/floodlightdefault.properties):

* net.floodlightcontroller.odin.master.OdinMaster.poolFile

```
# Pool-1
  NAME pool-1
  NODES 172.16.250.33 172.16.250.175
  NETWORKS odin
  APPLICATIONS net.floodlightcontroller.odin.applications.OdinMobilityManager

# Pool-2
  NAME pool-2
  NODES 172.16.250.29 172.16.250.37 
  NETWORKS guest-network
  APPLICATIONS net.floodlightcontroller.odin.applications.SimpleLoadBalancer
```
 
Each pool is defined by a name, a list of IP address of physical APs, the list of SSIDs or NETWORKS to be announced, and a list of applications that operate on that pool.

For testing purposes, if you'd like to assign a static IP to a client and have it connect to odin, you need to specify the client's details in a file pointed to by this property. This file is:

* net.floodlightcontroller.odin.master.OdinMaster.clientList

An example file looks as follows:

```
  00:16:7f:7e:00:00 172.16.250.2 00:1b:b3:7e:00:00 odin
  00:16:7f:7e:00:01 172.16.250.3 00:1b:b3:7e:00:01 odin
  00:16:7f:7e:00:02 172.16.250.4 00:1b:b3:7e:00:02 guest-network
```

Each row represents a client's MAC address, its static IP address, its LVAP's BSSID, and the SSID that its LVAP will announce.

To run the master:

```
  $: java -jar floodlight.jar
```

In our testbed the odin-agents are containerized using docker. xDPd runs in the same machine as the odin-agent but only the latest is containerized. The script setup-odin.sh (https://github.com/fgg89/odin-utilities/blob/master/scripts/setup-odin.sh) configures all the dependencies and starts the odin-agent and xDPd. Make sure you give the right parameters to this script. If you are using Ubuntu 14.04 you will need to set the wlan interface in ap mode. A script using hostapd (master-iface.sh) has been created for that purpose, it only needs to be run once and the configuration will remain until the machine is rebooted.
