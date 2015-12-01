# IP Floater
IP Floater tries to help to implement OpenStack-like floating IPs for general purpose.

## What is?
On one side, in an on-premises Cloud we may have a lot of private IP addresses but we usually have few public IP addresses. On the other side, not all the VMs in a Cloud need a public accessible IP address, as they usually have a gateway for registered users.

IP Floater consists of a server that enables associating a public accessible IP address managed by the IP Floater host to a private IP in the LAN. Later this IP can be associated to other private IP in the LAN.

## Use case
The use case is shown in the figure.
![IMAGE](https://github.com/dealfonso/ipfloater/blob/devel/img/ipfloater.jpg?raw=true =200x)

In that use case, we have multiple Virtual Machines, but in particular we have a web server (with private IP 192.168.1.40) and the front-end of a virtual cluster that is connected to its working nodes (with private IP 192.168.1.32). We need to access the web server and the front-end from the internet, but we do not need to access to the working nodes. The most common way is to access to the working nodes from the front-end.

In case that we had two public IP addresses (216.58.211.227 and 216.58.211.228) IPFloater could assign them to the private IP addresses and the private IPs will act as if they were these public IPs.

## Why?
OpenStack implements a mechanism of Floating IP addresses that (in brief) consist in a set of Public IP addresses that can be associated to private IP addresses. This is very similar to the IP address mechanism introduced in Amazon EC2.

But such Floating IPs are not available for a general case or other platforms such as OpenNebula.
## How?
IP Floater is based in iptables and implements pretty much the same rules that are implemented by the OpenStack's Floating IP, to make that the router host redirect the traffic directed to a public IP to a private IP in the LAN.

# Install

## Requirements
IPFloater has to be installed in a server that is connected to the internet and to the internal LAN that in which are the private IPs.

We'll assume eth0 for the interface that has access to the public network and eth1 for the interface that has access to the private network.

### Installing

```bash
$ sudo apt-get update && apt-get install python python-pip iptables git
$ sudo pip install --upgrade python-iptables cpyutils
$ sudo git clone https://github.com/dealfonso/ipfloater
$ cd ipfloater
$ sudo python setup.py install --record installed-files.txt
```

Now you have to create a configuration file in /etc/ipfloaterd.conf. You can start from the /etc/default/ipfloaterd.conf file.

```bash
$ cp /etc/default/ipfloaterd.conf /etc/ipfloaterd.conf
```

And you must edit the IP_POOL variable to set the comma separated pool of IP addresses.

Finally you can start the daemon:
```bash
$ sudo service ipfloaterd start
```
# Using IPFloater

IPFloater has a command line application whose help is self-contained
```
This is the client for ipfloaterd, which is a server that deals with iptables to enable floating IPs in private networks

Usage: ipfloater [-h] [--server-ip <value>] [--server-port <value>] [getip|releaseip|status|version|ippool]

	[-h|--help] - Shows this help
	[--server-ip|-i] <value> - The ip adress in which ipfloater listens
	[--server-port|-p] <value> - The ip port in which ipfloater listens
	* Requests a floating IP for a private IP
	  Usage: getip <ip>
		<ip> - private ip address to which is requested the floating ip

	* Releases the floating IP to a private IP
	  Usage: releaseip <ip>
		<ip> - private ip address to which is granted the floating ip

	* Gets the status of the redirections
	  Usage: status

	* Gets the version of the client and the server
	  Usage: version

	* Gets the public ip addresses in the pool
	  Usage: ippool
```
