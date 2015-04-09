# vbox-to-tor: Connector of Virtual Box machine to Tor for FreeBSD

This is the simple FreeBSD service that allows to seemlessly connect the VirtualBox machines to tor anonymity network.

## Installation

To install vbox-to-to simply copy the file:<br/>
cp vbox-to-tor /usr/local/etc/rc.d/

And add the following section to system configuration file /etc/rc.conf:<br/>
```shell
#
# for VBox VMs to TOR connection
#
firewall_enable="YES"
firewall_type="open"
vbox_tor_ifaces="tap0 tap1 tap2"
vbox_to_tor_enable="YES"
vbox_to_tor_ifaces="${vbox_tor_ifaces}"
vbox_to_tor_net_pattern="172.16"
tiny_dhcp_server_enable="YES"
tiny_dhcp_server_ifaces="${vbox_tor_ifaces}"
```

This allows you to run 3 different virtual machines connected to TOR.

