# vm-to-tor: Connect virtual machines (VirtualBox, bhyve, etc) to Tor on FreeBSD (FreeBSD port security/vm-to-tor)

This is the FreeBSD service that allows to seamlessly connect any number of the VirtualBox machines to the Tor anonymity network. (https://www.torproject.org/)

## Screenshot

![Alt text](https://raw.githubusercontent.com/yurivict/vm-to-tor/master/screenshot.png "Running with several VMs")

## Installation

vm-to-tor is installed by running this command as root:
```shell
pkg install vm-to-tor
```

## Running

### Setup the service

Add the following section to the system configuration file /etc/rc.conf:<br/>
```shell
#
# For VirtualBox VMs to Tor connections
#
firewall_enable="YES"
firewall_type="open"
vbox_tor_ifaces="tap0 tap1"
vm_to_tor_enable="YES"
vm_to_tor_ifaces="${vbox_tor_ifaces}"
```

This setup allows you to run 2 different virtual machines connected to TOR (on tap0 and tap1 tunnels).

Alternatively, add this line to /etc/rc.conf:<br/>
```shell
. /usr/local/etc/vm-to-tor.rc.conf.simple
```
and adjust /usr/local/etc/vm-to-tor.rc.conf.simple for your needs.

### Run the service

In order to run vm-to-tor, you need to execute these commands as root:
```shell
sysrc vm_to_tor_enable="YES"
service vm-to-tor start
```

### Setup in the Virtual Box

Click on "Settings" for the virtual machine that you want to connect to the Tor betwork.
Click on "Network", choose "Bridged Adapter" with one of the tapN interfaces.

## How does vm-to-tor work?

vm-to-tor creates tunnel interfaces and sets them up for the use by the virtual machines, so that all network traffic of the VMs is tunneled to the host level network interface. It also adds appropriate firewall rules that send all network traffic originating in the VMs directly to Tor router running on the host. These firewall rules also prevent any leaks of traffic through the original unsecured network connection on the host.

## Background

Tor Project actively promotes the so-called "Tor Browser Bundle" (TBB). The security of TBB depends on the vast and complex codebase that firefox browser is built from being free of bugs. Many bugs have been found in the firefox code in the past, particularly some severe JavaScript bugs that allowed for some breaches of anonymity of TBB users in the past. There is also no guarantee that more bugs will not be found. That is why many TBB users believe that it is much safer to disable JavaScript altogether, and thet they should limit what websites they should visit.

vm-to-tor took very different approach, "security by isolation". It allows user to easily connect the OS of his/her choice (VM guest) to the Tor router that runs on the host. This guest OS is completely isolated from the host and other guests, and can only connect to the Tor network. Therefore, regardless of what bugs any particular program that runs in the guest might have, such bugs just can't reveal the real IP of the user because anything they do always goes only through the Tor connection through a very simple tunnel.

There is another product, Whonix (https://www.whonix.org), that also chose security-by-isolation approach. However, Whonix requires the second VM to act as a Tor router host.

While providing an excellent security, security-by-isolation approach didn't become very popular primarily due to the setup difficulties. It requires virtual machines being setup in particular way, firewall rules, Tor setup. vm-to-tor eventually made such setup very easy for FreeBSD users.

vm-to-tor works with virtually no overhead, and installs as two standard FreeBSD services that can be started any time. It allows to run any number of guests, all completely isolated from each other and from the host.

## Caveats

* Stopping vm-to-tor while VMs are running currently causes all involved VMs to crash. I believe this is a bug in VirtualBox, but this isn't a very important problem.
* Changing networking type to tapN while VM is running also causes VM crash. This is another bug in VirtualBox.
* Programs requiring UDP will not work because Tor currently doesn't support UDP. Only DNS UDP is supported.

## Donations

We would appreciate donations: 1LDxJDTPkRS4RrPbzbYYPFr15sqmMZuJj5

