# vbox-to-tor: Connect VirtualBox machines to TOR on FreeBSD

This is the FreeBSD service that allows to seamlessly connect any number of the VirtualBox machines to the TOR anonymity network. (https://www.torproject.org/)

## Installation (in less than 15 seconds)

You need to have the ports tree installed. Here are all commands you need to install vbox-to-tor, execute them as root:
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

In order to run it you need to have both VirtualBox and TOR installed and running, and execute these commands as root:
```shell
/usr/local/etc/rc.d/vbox-to-tor start
/usr/local/etc/rc.d/tiny-dhcp-server start
/usr/local/etc/rc.d/tor restart
```
Choose "Bridged Adapter" with one of the tapN interfaces for the VM you want to connect to TOR.

## Installation (step by step)

To install vbox-to-tor itself copy the file:<br/>
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
tiny_dhcp_server_enable="YES"
tiny_dhcp_server_ifaces="${vbox_tor_ifaces}"
```

vbox-to-tor depends on this DHCP server: https://github.com/yurivict/tiny-dhcp-server. To install it execute these commands (which will require the ports tree):
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

## How does vbox-to-tor work?

vbox-to-tor creates tunnel interfaces and sets them up for the use by the virtual machines, so that all network traffic of the VMs is tunneled to the host level network interface. It also adds appropriate firewall rules that send all network traffic originating in the VMs directly to TOR router running on the host. These firewall rules also prevent any leaks of traffic through the original unsecured network connection on the host.

## Background

Tor Project actively promotes the so-called "Tor Browser Bundle" (TBB). The security of TBB depends on the vast and complex codebase that firefox browser is built from being free of bugs. Many bugs have been found in the firefox code in the past, particularly some severe JavaScript bugs that allowed for some breaches of anonymity of TBB users in the past. There is also no guarantee that more bugs will not be found. That is why many TBB users believe that it is much safer to disable JavaScript altogether, and thet they should limit what websites they should visit.

vbox-to-tor took very different approach, "security by isolation". It allows user to easily connect the OS of his/her choice (VM guest) to the TOR router that runs on the host. This guest OS is completely isolated from the host and other guests, and can only connect to the TOR network. Therefore, regardless of what bugs any particular program that runs in the guest might have, such bugs just can't reveal the real IP of the user because anything they do always goes only through the TOR connection through a very simple tunnel.

There is another product, Whonix (https://www.whonix.org), that also chose security-by-isolation approach. However, Whonix requires the second VM to act as a TOR router host.

While providing an excellent security, security-by-isolation approach didn't become very popular primarily due to the setup difficulties. It requires virtual machines being setup in particular way, firewall rules, TOR setup. vbox-to-tor eventually made such setup very easy for FreeBSD users.

vbox-to-tor works with virtually no overhead, and installs as two standard FreeBSD services that can be started any time. It allows to run any number of guests, all completely isolated from each other and from the host.

## Caveats

* For better experience with vbox-to-tor you need kernel with this patch: https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=199136 Without this patch tapN interfaces will be brought down with every guest reboot, and you will need to bring them up manually every time.
* Stopping vbox-to-tor while VMs are running currently causes all involved VMs to crash. I believe this is a bug in VirtualBox, but this isn't a very important problem.
* Changing networking type to tapN while VM is running also causes VM crash. This is another bug in VirtualBox.
* Programs requiring UDP will not work, because TOR currently doesn't support UDP. Only DNS UDP is supported.
* vbox-to-tor writes torrc files directly due to the lack of the privileged auto-authentication feature in TorCtrl protocol. Normally TOR doesn't self-modify torrc, but if any other programs (ex. arm or vidalia) would modify TOR config, vbox-to-tor changes to torrc can be either lost or impossible to remove for vbox-to-tor (see TOR ticket#15649)

## Ports

I am willing to create corresponding FreeBSD ports if there is sufficient user interest.

## Donations

We would appreciate donations: 1LDxJDTPkRS4RrPbzbYYPFr15sqmMZuJj5

