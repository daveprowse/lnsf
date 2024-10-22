# Part II: Network Commands & Configurations

There are dozens of commands that deal with Linux networking. While it is difficult to know them all (and their options), there are a few basic ones that just about every tech should know. We'll cover those in this section.

## *ip and ping*

The ip and ping commands are two of the most basic analysis tools in Linux. They can be helpful when obtaining information, troubleshooting, and making network connections. For these labs you can use any Linux distribution.

> Remember: If you need more information about any command, to use the help file (for example `ip --help`, or simply `ip -h`) or the manual page (for example `man ip`). Use the help and manual pages for any command that you need more information about. 

### Lab 2-1

**Analyzing with the ip command**

- Find out Ethernet-based information. Type `ip link`

	Below are results of the ip link command run on a Debian client:

	```
	root@deb52:~# ip link
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
	    link/ether 52:54:00:6d:fb:ac brd ff:ff:ff:ff:ff:ff
	3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
	    link/ether 52:54:00:69:db:db brd ff:ff:ff:ff:ff:ff
	4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN mode DEFAULT group default qlen 1000
	    link/ether 52:54:00:69:db:db brd ff:ff:ff:ff:ff:ff
	```

	From this list we can see the ethernet-based information of each network connection on the system. For example, the 2nd connection *enp1s0* has a MAC address of 52:54:00:6d:fb:ac. The MAC address is normally 6 octets and is colon-separated. This address is burned into the PROM chip of the network interface, or (in the case of virtual machines) it is assigned to a virtual network interface. We also see that *enp1s0* is "UP", meaning that it is activated and ready to transmit and receive data.  

- Discover IPv4 and IPv6 addresses. Type `ip a`

	Below are results of the ip a command run on a Debian client:

	```
	root@deb52:~# ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	    inet6 ::1/128 scope host 
	       valid_lft forever preferred_lft forever
	2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 52:54:00:6d:fb:ac brd ff:ff:ff:ff:ff:ff
	    inet 10.0.2.52/24 brd 10.0.2.255 scope global noprefixroute enp1s0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::36b4:f533:4478:39c8/64 scope link noprefixroute 
	       valid_lft forever preferred_lft forever
	3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
	    link/ether 52:54:00:69:db:db brd ff:ff:ff:ff:ff:ff
	    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
	       valid_lft forever preferred_lft forever
	4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN group default qlen 1000
	    link/ether 52:54:00:69:db:db brd ff:ff:ff:ff:ff:ff
	```

	Now, in addition to the inforamtion shown with the ip link command, we can see the associated IPv4 and IPv6 addresses. For example, the 2nd interface (*enp1s0) hsa the IPv4 address 10.0.2.52/24. However, this command shows a lot of information. And for a system with a lot of network connections, it can quickly become unmanageable. So, we can sort and filter the information as we see fit.

- Sort and filter using the ip command

	Example 1:  `ip -br a`

		```
		root@deb52:~# ip -br a
		lo               UNKNOWN        127.0.0.1/8 ::1/128 
		enp1s0           UP             10.0.2.52/24 fe80::36b4:f533:4478:39c8/64 
		virbr0           DOWN           192.168.122.1/24 
		virbr0-nic       DOWN        
		```

		With this command we get a nice table of information telling us the state of the interface and the IP addresses. I use it often!

	Example 2:  `ip a | grep inet | sort -n`

		```
		root@deb52:~# ip a | grep inet | sort -n
		    inet 10.0.2.52/24 brd 10.0.2.255 scope global noprefixroute enp1s0
		    inet 127.0.0.1/8 scope host lo
		    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
		    inet6 ::1/128 scope host 
		    inet6 fe80::36b4:f533:4478:39c8/64 scope link noprefixroute 
		```
		
		With this command we filter for the term "inet" and sort the results numerically. You could also add a "column -t" to make a nice table out of it. 

	Example 3:  `ip --oneline addr | column -t`
		
    ```
		root@deb52:~# ip --oneline addr | column -t
		1:  lo      inet   127.0.0.1/8                   scope  host             lo\            valid_lft  forever        preferred_lft  forever
		1:  lo      inet6  ::1/128                       scope  host             \              valid_lft  forever        preferred_lft  forever
		2:  enp1s0  inet   10.0.2.52/24                  brd    10.0.2.255       scope          global     noprefixroute  enp1s0\        valid_lft      forever        preferred_lft  forever
		2:  enp1s0  inet6  fe80::36b4:f533:4478:39c8/64  scope  link             noprefixroute  \          valid_lft      forever        preferred_lft  forever
		3:  virbr0  inet   192.168.122.1/24              brd    192.168.122.255  scope          global     virbr0\        valid_lft      forever        preferred_lft  forever
		```
		
		With this command we show each IPv4 and IPv6 interface as one line of information. 

	You can imagine the possibilities when it comes to a command like ip. Search for, and practice with different options to make your work more efficient, and easier on the eyes. 

### Lab 2-2
**Adding and removing IP addresses with the ip command**

- To add an IP address, use the following syntax: `ip a add <ip_address> dev <interface>`. So for example:

	`ip a add 10.0.2.152/24 dev enp1s0`

	This would add the IP address 10.0.2.152 to the interface named *enp1s0*. However, this is meant for temporary purposes. If you need a persistent change then another tool is recommended, for example, nmcli.

- To remove an IP address, simple change *add* to *delete*. For example:

	`ip a delete 10.0.2.152/24 dev enp1s0`

### Lab 2-3
**Working with the `ip route` command**

- To view the default gateway, type `ip route show` or `ip route` or simply `ip r`. Example:

	```
	root@deb52:~# ip r
	default via 10.0.2.1 dev enp1s0 proto static metric 100 
	10.0.2.0/24 dev enp1s0 proto kernel scope link src 10.0.2.52 metric 100 
	192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
	```

	In the example, the default gateway is shown on the second line as 10.0.2.1. Any networks that this system has access to are also shown. In the example we have 10.0.2.0 and 192.168.122.0. To make columns, simply add the column -t option: `ip r | column -t`. (This is similar to the older `route -n` command if using the net-tools package.) If you have a lot of routes, you can filter for the default gateway easily by typing: `ip r | grep default`.

- Remove and put back the default route (or gateway). 

	In the following example, we use the command `ip r delete default` to remove the default route, then the `ip r` command to check our work, then to put it back: `ip r add default via 10.0.2.1`

	```
	root@deb52:~# ip r delete default
	root@deb52:~# ip r
	10.0.2.0/24 dev enp1s0 proto kernel scope link src 10.0.2.52 metric 100 
	192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
	root@deb52:~# ip r add default via 10.0.2.1
	root@deb52:~# ip r
	default via 10.0.2.1 dev enp1s0 
	10.0.2.0/24 dev enp1s0 proto kernel scope link src 10.0.2.52 metric 100 
	192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
	```

- Add and remove a route.

	To add a new route, use the following syntax: `ip r add <ip_network> dev <interface>`. View the new route with the `ip r` command. 
	
	To remove that route, change "add" to "delete". Example below:

	```
	root@deb51:~# ip r add 172.21.0.0/16 dev enp1s0
	root@deb51:~# ip r
	default via 10.0.2.1 dev enp1s0 onlink 
	10.0.2.0/24 dev enp1s0 proto kernel scope link src 10.0.2.51 
	172.21.0.0/16 dev enp1s0 scope link 
	root@deb51:~# ip r delete 172.21.0.0/16 dev enp1s0
	root@deb51:~# ip r
	default via 10.0.2.1 dev enp1s0 onlink 
	10.0.2.0/24 dev enp1s0 proto kernel scope link src 10.0.2.51 
	```

### Lab 2-4
**Working with the `ping` command**

Ping is the most primal testing command. It can show you if your TCP/IP stack is working properly locally. It can tell you if you have access to the default gateway. It can tell you if other systems are "up", or at least, if they are accessible from your location. Here are some examples:
	
Example: Pinging the localhost on IPv4
	
  ```
	root@deb52:~# ping -4 127.0.0.1
	PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
	64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.056 ms
	64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.069 ms
	64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.067 ms
	^C
	--- 127.0.0.1 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 52ms
	rtt min/avg/max/mdev = 0.056/0.064/0.069/0.005 ms
	```

	With this command we ping the local system on its IPv4 loopback address: 127.0.0.1. The system received three 64 byte replies before I cancelled the operation by pressing Ctrl + c. You could also ping the localhost by name (for example *deb52*) or via IPv6 which would be ::1

Example: Pinging the gateway address
	
  ```
	root@deb52:~# ping 10.0.2.1
	PING 10.0.2.1 (10.0.2.1) 56(84) bytes of data.
	64 bytes from 10.0.2.1: icmp_seq=1 ttl=64 time=0.126 ms
	64 bytes from 10.0.2.1: icmp_seq=2 ttl=64 time=0.236 ms
	64 bytes from 10.0.2.1: icmp_seq=3 ttl=64 time=0.226 ms
	^C
	--- 10.0.2.1 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 54ms
	rtt min/avg/max/mdev = 0.126/0.196/0.236/0.049 ms
 	```

	In this example, we pinged the gateway device which is 10.0.2.1. Notice the difference in the reply time. In the first example, it averaged 0.064 milliseconds (ms), and in the second example, it averaged 0.196 ms - substantially longer (but still quick). That's the difference between pinging the local system (which generated no network traffic) and pinging another system (even though the gateway is just a virtual device). If I was to ping my actual physical gateway, the reply times would average around 0.5 ms. 

Example: Pinging a host on the Internet
	
  ```
	root@deb52:~# ping prowse.tech
	PING prowse.tech (67.205.11.189) 56(84) bytes of data.
	64 bytes from apache2-igloo.allatou.dreamhost.com (67.205.11.189): icmp_seq=1 ttl=49 time=30.7 ms
	64 bytes from apache2-igloo.allatou.dreamhost.com (67.205.11.189): icmp_seq=2 ttl=49 time=20.8 ms
	64 bytes from apache2-igloo.allatou.dreamhost.com (67.205.11.189): icmp_seq=3 ttl=49 time=20.4 ms
	^C
	--- prowse.tech ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 6ms
	rtt min/avg/max/mdev = 20.406/23.959/30.717/4.780 ms
  ```
    
  In this example, we pinged "prowse.tech". Because the ping was against a name, that name had to be resolved to its IP address (67.205.11.189). Also note that the reply times are much higher, averaging 23.9 ms per reply. Ping rates of 20 - 100 ms are common when connecting to websites. 

Example: Modifying the ping packet size and amount of replies

	```
	root@deb52:~# ping -c 5 -s 1024 10.0.2.1
	PING 10.0.2.1 (10.0.2.1) 1024(1052) bytes of data.
	1032 bytes from 10.0.2.1: icmp_seq=1 ttl=64 time=0.136 ms
	1032 bytes from 10.0.2.1: icmp_seq=2 ttl=64 time=0.226 ms
	1032 bytes from 10.0.2.1: icmp_seq=3 ttl=64 time=0.219 ms
	1032 bytes from 10.0.2.1: icmp_seq=4 ttl=64 time=0.210 ms
	1032 bytes from 10.0.2.1: icmp_seq=5 ttl=64 time=0.232 ms
	
	--- 10.0.2.1 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 102ms
	rtt min/avg/max/mdev = 0.136/0.204/0.232/0.038 ms
	```
  
  This time we added two options. `-c (count)` specifies the exact amount of ping requests (and hopefully, replies) - in this case I selected 5. You can see that I didn't have to press Ctrl + c to break out of the ping process. `-s (packetsize)` allows you to change the ICMP echo - I changed it from the default 64 bytes to 1024 bytes. The replies add 8 bytes of supervisory information, bringing it to 1032 bytes per reply packet. 
  
  This type of ping modification can be helpful when testing servers and attempting to simulate more traffic, or conducting longer tests with a specific amount of pings. The maximum packet size on most versions of Linux is 65507 bytes, which creates fragmented information, and could be blocked by a security device, so the maximum recommended testing amount is around 1400 bytes. (This keeps it within the realm of normal 1500 byte IP packets).  

> Note: Sometimes you might want an automated ping script that can check if multiple hosts are up at the same time. I have one for you called "Superping" on GitHub at [this link.](https://github.com/daveprowse/lnsf){target=_blank} 

## ***Hostname and DNS***

We've done plenty up to this point with IP addresses. But most systems (and people) communicate by name. So a technician needs to know how to modify hostnames and configure DNS. Remember that the Domain Name System (DNS) is in charge of resolving domain names (and host names) to their respective IP addresses - and vice-versa. 

### Lab 2-5
**Working with `hostnamectl`**

- View system information with the `hostnamectl` command

	Type `hostnamectl` to see the hostname and other information about the system. Example:

	```
	root@deb52:~# hostnamectl
	  Static hostname: deb52
	        Icon name: computer-vm
	          Chassis: vm
	         Location: workplace2
	       Machine ID: 42f779dc9123405c90f8fa73b6c83f7c
	          Boot ID: 35b9c976794a4bb38ba7a4e81511152a
	   Virtualization: kvm
	 Operating System: Debian GNU/Linux 10 (buster)
	           Kernel: Linux 4.19.0-16-amd64
	     Architecture: x86-64
	```

	Here we see the name of the computer is *deb52*. We get a whole lot of other information as well, including the operating system type and version, the version of the Linux kernel, and the architecture of the computer. 

- Use the `hostnamectl` command to change the hostname. 
	- First, change the name by typing `hostnamectl set-hostname newname`
	- View the new name by entering `hostnamectl`
	- Close the terminal and open a new one to see the new name in the prompt.
	- Change the name back to the original.
	- Close the terminal and open a new one again.
	- Verify the name is back to the original. (You can also use the older `hostname` command to see the hostname only.)

	Example: (the four dashes represents closing the terminal and reopening it)

	```
	root@deb52:~# hostnamectl set-hostname newname
	root@deb52:~# hostnamectl
	   Static hostname: newname
	         Icon name: computer-vm
	           Chassis: vm
	          Location: workplace2
	        Machine ID: 42f779dc9123405c90f8fa73b6c83f7c
	           Boot ID: 35b9c976794a4bb38ba7a4e81511152a
	    Virtualization: kvm
	  Operating System: Debian GNU/Linux 10 (buster)
	            Kernel: Linux 4.19.0-16-amd64
	      Architecture: x86-64
	----
	root@newname:~# hostnamectl set-hostname deb52
	----
	root@deb52:~# hostname
	deb52
	```

	> Note: A fun program that shows similar information to hostnamectl is called neofetch. Install it by name with your distro's installer - for example `apt install neofetch`. Then run the program by simply typing `neofetch`. 

### Lab 2-6
**Viewing the DNS configuration**

You can view the DNS configuration in different ways depending on the distribution and the networking service used. 

- View the DNS server setting with resolv.conf:

	Type `cat /etc/resolv.conf` . The results will look similar to the example below.
	```
	root@deb52:~# cat /etc/resolv.conf
	# Generated by NetworkManager
	nameserver 10.0.2.1
	```

	In this example I used a Debian client. That system uses NetworkManager, which generates the DNS (or nameserver) information and places it in the resolv.conf file. To view/modify the DNS server IP address on a system that runs NetworkManager you would use the nmcli command. 

- View the DNS server setting with the nmcli command:

	To view the DNS server setting with nmcli, simply type `nmcli` and look toward the end of the results for "DNS configuration. Here's an example:

	```
	DNS configuration:
	        servers: 10.0.2.1
	        interface: enp1s0
	```

	To modify the DNS coniguration you could use syntax similar to this:

	`nmcli connection modify enp1s0 ipv4.dns 10.0.2.3`

	That would change the DNS server setting from 10.0.2.1 to 10.0.2.3. We'll cover nmcli more in an upcoming section.

- View the DNS server setting with resolved.conf

	If a system uses the systemd-networkd network configuration, then the settings are stored in /etc/systemd/resolved.conf. For example:

	```
	[Resolve]
	DNS=10.0.2.1
	FallbackDNS=10.0.2.2
	#Domains=
	#LLMNR=yes
	#MulticastDNS=yes
	#DNSSEC=allow-downgrade
	#DNSOverTLS=no
	#Cache=yes
	#DNSStubListener=yes
	#ReadEtcHosts=yes
	```

 	In this example, we have a primary DNS server (10.0.2.1) and a fallback, or secondary, DNS server (10.0.2.2). resolved.conf is less commonly used. Though Ubuntu server uses systemd-networkd, it relies on netplan as the front-end for the configuration. For example:

	```
	# This is the network config written by 'subiquity'

	network:       
	  version: 2
	  renderer: networkd
	  ethernets:
	    enp1s0:
	      addresses: [10.0.2.53/24]
	      gateway4: 10.0.2.1
	      nameservers: 
	        search: [example.com]
	        addresses: [10.0.2.1]
	```

	Here, the DNS servers are known as "nameservers" and there is one listed at 10.0.2.1.

	!!! note
		If you were working at a Debian server with a default configuration, you would simply use resolv.conf, and would modify DNS there. This is because it uses the networking service, so networkd and NetworkManager do not apply. 

## ***nmcli***

nmcli is the NetworkManager command-line interface. This is a very commonly used command line tool for analyzing, modifying, and troubleshooting network connections. It is used by Fedora/RHEL/CentOS, Debian (as a client) Ubuntu Desktop, and many more Linux distributions. It can be used as a single-command tool or within an interactive interface. We'll start with the nmcli command in the command line. Keep in mind - this command is boss!

### Lab 2-7
**View and analyze the network configuration with nmcli.**

- Basic usage of nmcli.

	Type `nmcli` to see your network configuration. Example results on a Debian client are below. Results on other systems running NetworkManager should be very similar. 

	Example of the `nmcli` command on a Debian client
		
    ```
		enp1s0: connected to Wired connection 1
		"Red Hat Virtio"
		ethernet (virtio_net), 52:54:00:6D:FB:AC, hw, mtu 1500
		ip4 default
		inet4 10.0.2.52/24
		route4 10.0.2.0/24
		route4 0.0.0.0/0
		inet6 fe80::36b4:f533:4478:39c8/64
		route6 fe80::/64

		virbr0: connected to virbr0
		"virbr0"
		bridge, 52:54:00:69:DB:DB, sw, mtu 1500
		inet4 192.168.122.1/24
		route4 192.168.122.0/24

		lo: unmanaged
		"lo"
		loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

		virbr0-nic: unmanaged
		"virbr0-nic"
		tun, 52:54:00:69:DB:DB, sw, mtu 1500

		DNS configuration:
		servers: 10.0.2.1
		interface: enp1s0

		Use "nmcli device show" to get complete information about known devices and
		"nmcli connection show" to get an overview on active connection profiles.

		Consult nmcli(1) and nmcli-examples(5) manual pages for complete usage details.
    ```

    At the beginning of the results we see "enp1s0: connected to "Wired connection 1". *enp1s0* is the Linux hardware-based name for the network interface. But NetworkManager gives these devices its own names - in this case, *Wired connection 1*. That is the name that we need to use when configuring the network interface with the nmcli command. Lets show how to add and remove static and DHCP-based IP addresses. 

- View the NetworkManager connections

	Type `nmcli connection show`. Here's an example:

	```
	[sysadmin@smauggy ~]$ nmcli connection show
	NAME                UUID                                  TYPE      DEVICE 
	Wired connection 1  58f4b9c3-638d-31e4-bdbf-010a3b56bf47  ethernet  enp3s0 
	EMF-5B              257a1403-e9fd-4a05-bb5c-e91f7baf5274  wifi      wlp2s0 
	virbr0              12a61b1b-f6c6-42a1-b62c-7777cfb94763  bridge    virbr0 
	```

	This example was taken from an actual laptop with wired and wireless connections. Under the "TYPE" column you can see there is an "ethernet" device named *enp3s0*. That is the wired connection, and so NetworkManager calls is "Wired connection 1". Under "TYPE" you will also see a "wifi" device named *wlp2s0*. In this case, NetworkManager refers to it by the name "EMF-5B" (which is actually the name of the wireless network it is connect too - a bit of a security issue, but one which is fixable).

	> Note: You can abbreviate (or truncate) nmcli commands a lot. For example, `nmcli connection show` can be reduced to `nmcli con show`, or just `nmcli c show`. In fact, the "show" portion isn't even necessary. So you could just type `nmcli c` and be done with it! You'll get the same results.

### Lab 2-8
**Working with nmcli in the command line**

- Add an IP address with nmcli. Example:
	
  ```
	nmcli connection modify "Wired connection 1" ipv4.method manual ipv4.address 10.0.2.152/24 ipv4.gateway 10.0.2.1 ipv4.dns 10.0.2.1
	```

	In this example, we specify that the address to be added will be static, then we add the IP address 10.0.2.152/24, and then we add the gateway and DNS IP addresses. You can add multiple IPs if you needed to in this manner, just by comma separating them:
	
  ```
	nmcli connection modify "Wired connection 1" ipv4.method manual ipv4.address 10.0.2.152/24,10.0.2.153/24
	```

	> Note:	You can abbreviate here too, for example: `nmcli con mod` or even `nmcli c m` instead of `nmcli connection modify`. 

		Combine this with tab completion. For example, for the network interface type `"W` and press the tab key. That will auto-complete the name of the interface, which in this case is "Wired connection 1". Combine auto-complete with abbreviations and it can be a real time-saver. Wonderful!

- Down and up the interface

	Once you have made your changes, you need to deactivate and reactivate the network interface for the changes to take effect. This is known as "down" and "up" the interface. To do this type the following two commands:
	`nmcli connection down "Wired connection 1"`
	and
	`nmcli connection up "Wired connection 1"`

	At that point you should see the new IP addresses listed when you run the nmcli command. The following example shows a snippet of the nmcli results. 

	```
	enp1s0: connected to Wired connection 1
	"Red Hat Virtio"
	ethernet (virtio_net), 52:54:00:6D:FB:AC, hw, mtu 1500
	ip4 default
	inet4 10.0.2.152/24
	inet4 10.0.2.153/24
	route4 10.0.2.0/24
	```

	You can see the two IP addresses that were added previously.

- Set the interface to obtain an IP address from a DHCP server.

	Type `nmcli c m "Wired connection 1" ipv4.method auto`

	By selecting "auto" we set the interface to obtain all TCP/IP information from a DHCP server (if one is available) including it's IP address, netmask, gateway address, and DNS server IP address. Down and up the interface for the changes to take effect. You should see something similar to the example snippet below:

	```
	enp1s0: connected to Wired connection 1
	"Red Hat Virtio"
	ethernet (virtio_net), 52:54:00:6D:FB:AC, hw, mtu 1500
	ip4 default
	inet4 10.0.2.152/24
	inet4 10.0.2.153/24
	inet4 10.0.2.139/24
	route4 10.0.2.0/24
	```

	In this case, the system obtained an IP address from a DHCP server on my virtual network. It received the address 10.0.2.139. Now we have two static IP addresses and one dynamic IP address!

- Remove one of the static IP addresses.

	Type `nmcli c m "Wired connection 1" -ipv4.address 10.0.2.153/24` Note the - dash before ipv4, and type in an appropriate IP address based on your configuration. Down and up the interface and you should see the results with the nmcli command. 

### Lab 2-9
**Working with the nmcli interactive shell**

- Access the nmcli shell.

	You can access the nmcli interactive shell for any one of your network interfaces. Once there, you can run multiple commands, and save them all at once. To access the nmcli shell for an interface enter the following:

	`nmcli connection edit "Wired connection 1"`

	Note that it says "edit" this time. That is the option that opens a shell. Remember to change the interface name based on your system. You should see something similar to the following:

	```
	root@deb52:~# nmcli connection edit "Wired connection 1" 

	===| nmcli interactive connection editor |===

	Editing existing '802-3-ethernet' connection: 'Wired connection 1'

	Type 'help' or '?' for available commands.
	Type 'print' to show all the connection properties.
	Type 'describe [<setting>.<prop>]' for detailed property description.

	You may edit the following settings: connection, 802-3-ethernet (ethernet), 802-1x, dcb, sriov, ethtool, match, ipv4, ipv6, tc, proxy
	nmcli> 
	```

	Now, we can enter commands into the shell. 

- Remove the other static IP address. 

	In the shell, type `remove ipv4.address 10.0.2.152/24`. This will remove the static IP address. 

- Set a new static IP address.

	Type `set ipv4.address 10.0.2.52/24`. This will set the original IP address that the system had before. If it asks, type "yes" to set the IP address to manual. 

	!!! note
		You can also abbreviate here. Instead of "remove" type "r", and instead of "set", type "s". Every character counts!

- Save the configuration, activate it, and quit.

	To save the configuration, simply type `save`. Then type `activate` to enable it. Finally, type `quit` to exit out of the shell.  At this point, our IP configuration should be back to what it was when we started Lab 3-7. 

	The beauty of the nmcli shell is that we can do multiple operations like this without having to type "nmcli" every time, or specifying the network interface everytime, because the shell we opened is dedicated to the interface we specified in the beginning. 
!!! note
	For more information about nmcli use the `nmcli -h` and `man nmcli` commands. 

	Also, see the following link:

	https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_nmcli
  
**Summary**: In this section we covered a good amount of commands. When it comes to knowing commands, its all about practice, and researching the help and man files. The more you practice, the more fluent you become with the command structure. 

> Tip: But remember this! You can't know all commands and all options for each command. It just won't happen. It's more important to know how to search for the information you seek!

We worked with the `ip` command which allows us to analyze Ethernet information, TCP/IP information, routing information, and more. We also worked with ping. Some admins don't like to use the tool, but because it is such an easy and primal tool, and because it just works, it becomes almost inescapable. Ping can be used for basic testing, troubleshooting, and to get a basic baseline of a system.

Then we moved into hostnames and DNS. You can change the hostname with the `hostnamectl` command or by accessing the /etc/hostname file. DNS can be modified within /etc/resolv.conf, or (if using networkd) /etc/resolved.conf, or (if using NetworkManager) with the nmcli command (among other places). 

The `nmcli` command is extremely powerful and well-versed. You can analyze systems, change TCP/IP information, work with it as a single command or in an interactive shell. One look at the MAN page shows you the depth of the command. That's one of those commands that requires a lot of practice to master - everyday. 

So that wraps up this part. Remember to practice the commands and options, but also, to know how to search for the commands and options that you need to get your work done quickly and efficiently. 

---
