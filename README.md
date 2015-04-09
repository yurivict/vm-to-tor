# vbox-to-tor: Connector of VirtualBox machines to TOR for FreeBSD

This is the FreeBSD service that allows to seamlessly connect any number of the VirtualBox machines to TOR anonymity network. (https://www.torproject.org/)

## Installation (in less than 15 seconds)

If you have the ports tree installed, all commands you need to achieve the same that is explained in the previous section:
```shell
cd /tmp
git clone https://github.com/yurivict/tiny-dhcp-server
cp tiny-dhcp-server/tiny-dhcp-server.py /usr/local/bin/tiny-dhcp-server
cp tiny-dhcp-server/tiny-dhcp-server /usr/local/etc/rc.d/
rm -rf tiny-dhcp-server
pkg install --automatic --no-repo-update python34
(cd /usr/ports/net/py-netifaces && PYTHON_VERSION=3.4 make install clean)
git clone https://github.com/yurivict/freebsd-vbox-to-tor
cp freebsd-vbox-to-tor/vbox-to-tor /usr/local/etc/rc.d/
cat freebsd-vbox-to-tor/rc.conf.sample >> /etc/rc.conf
rm -rf freebsd-vbox-to-tor
```

## Installation (step by step)

To install vbox-to-tor simply copy the file:<br/>
```shell
cp vbox-to-tor /usr/local/etc/rc.d/
```

Also add the following section to the system configuration file /etc/rc.conf:<br/>
```shell
#
# For VirtualBox VMs to TOR connections
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

vbox-to-tor also depends on this DHCP server: https://github.com/yurivict/tiny-dhcp-server. To install it execute these commands:
```shell
git clone https://github.com/yurivict/tiny-dhcp-server
cp tiny-dhcp-server/tiny-dhcp-server.py /usr/local/bin/tiny-dhcp-server
cp tiny-dhcp-server/tiny-dhcp-server /usr/local/etc/rc.d/
rm -rf tiny-dhcp-server
pkg install --automatic --no-repo-update python34
(cd /usr/ports/net/py-netifaces && PYTHON_VERSION=3.4 make install clean)
```

This setup allows you to run 3 different virtual machines connected to TOR (on tap0, tap1 and tap2 tunnels).

After this you need to choose "Bridged Adapter" as a networking adapter for VMs in VirtualBox Manager. You need to assign one of these tapN devices to the bridged interface of each VM you want to connect to TOR. As simple as that.

## How vbox-to-tor works?

vbox-to-tor creates tunnel interfaces and sets them up for the use by the virtual machines. It also adds appropriate firewall rules that tunnel all communication originating in the VM directly to TOR running on the host.


## Background

Tor project actively promotes the so-called "Tor Browser Bundle" (TBB). The security of TBB depends on the absence of bugs in the vast and complex codebase that firefox browser is built from. Many bugs have been found in the firefox code in the past, particularly some severe JavaScript bugs that allowed for some breaches of anonymity of TBB users in the past. There is also no guarantee that more bugs won't be found. That is why many TBB users believe that it is much safer to disable JavaScript altogether.

vbox-to-tor took very different approach, "security by isolation". It allows user to easily connect the OS of his/her choice (VM guest) to the TOR router that runs on the host. This guest OS is completely isolated from the host and other guests, and can only connect to the TOR network. Therefore, regardless of what bugs any particular program that runs in the guest might have, such bugs just can't reveal the real IP of the user because anything they do all goes only through the TOR connection through a very simple tunnel.

There is another product, Whonix (https://www.whonix.org), that also chose security-by-isolation approach. However, Whonix requires the second VM to act as a tor host.

vbox-to-tor works with virtually no overhead. It allows to run any number of guests, all completely isolated from each other and from the host.

## Caveats

For better experience with vbox-to-tor you need kernel with this patch: https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=199136 Without this patch tapN interfaces will be brought down with every guest reboot, and you will need to bring them up manually every time.


## Donations

We would appreciate donations: 1LDxJDTPkRS4RrPbzbYYPFr15sqmMZuJj5
