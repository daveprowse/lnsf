# Part III: Networking Services & Configuration Files
The core networking services that are used by many Linux distributions include the networking, networkd, and NetworkManager services. These are the underlying services for other networking services such as HTTP, DNS, DHCP, and so on. Normally, a system will only use one of the three at any given time.  The following three subsections include labs to work with each of the three.

## *The networking service*
The networking service is mainly used by Debian servers. Therefore, you will need a Debian server (or some other system using the networking service) to accomplish the labs in this section. 
 
### Lab 3-1 
**Use common commands associated with the networking service**

- Verify if the networking service is running:

	`systemctl status networking.service`

- Enable/disable the service: 

	`systemctl enable networking` and `systemctl disable networking`

- Start/stop the service:

	`systemctl start networking` and `systemctl stop networking`

- Enable and start the service with one command:

	`systemctl --now enable networking`

- Disable and stop the service with one command:

	`systemctl --now disable networking`

When you are finished, enable and start the networking service so that we can use the system normally. 

> Info:	For more information about the systemctl command, type `systemctl --help` and `man systemctl`

### Lab 3-2
**Configure networking and down and up the interface**

- Analyze the networking configuration:

	`ip a`

- View and modify the network configuration file. 

	It is located at: /etc/networking/interfaces

> Example:
    Here's an example of the interfaces file in Debian. You can use Debian's built-in text editor: Nano, but I prefer Vim. To install Vim, type `apt install vim`. 

    ```
    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    allow-hotplug enp1s0
    iface enp1s0 inet static
        address 10.0.2.51/24
        gateway 10.0.2.1
    ```

  If you configured your system to obtain an IP address during the installation of Debian, then it would show "dhcp" instead of "static", and there would be no address or gateway entries. But you should configure your server to use a static IP address based on your IP network as shown in the example. If you already have a static IP address, try changing the address to something different. Then, save and quit out of the file. If you are using Vim, save and quit by typing `:wq` and pressing enter. 
		
- Deactivate and reactivate the network interface:

	`ifdown <interface>` and `ifup <interface>`

	Here's a real example, performing both actions with a single command. In this case, the network interface is *enp1s0* but your interface name will most likely be different.

	`ifdown enp1s0 && ifup enp1s0`

- Test the connection:

	`ping example.com` or use whatever test site you wish. 

### Lab 3-3
**Verify the DNS configuration**

Most likely, the DNS configuration is correct. But we should check it anyway, and then test it.

- Analyze the DNS configuration:

    `cat /etc/resolv.conf`

    This should show an IP address of a DNS server. For example, our Debian server above shows the DNS server IP as 10.0.2.1. That is the IP address of the built in DNS forwarder in the virtualization system (it is the same as the gateway address). In some cases you might have to modify the address to something else - for example, the IP address of the router on your network, or some other device's address on the LAN. But if you are using a virtualization system (VirtualBox, Vmware, Hyper-V, KVM, etc...) then the DNS forwarder should be the first IP on your computer's IP network. So for example, the Debian server above is on the 10.0.2.0 network. It has an IP of 10.0.2.51 and uses the virtualization system's IP of 10.0.2.1 for DNS and the gateway.

- Test if DNS is resolving properly:

    `ping prowse.tech`

    This should translate the name *prowse.tech* to its current IP address. For example:
    ```
    root@deb51:~# ping prowse.tech
    PING prowse.tech (67.205.11.189) 56(84) bytes of data.
    64 bytes from apache2-igloo.allatou.dreamhost.com (67.205.11.189): icmp_seq=1 ttl=49 time=21.9 ms
    64 bytes from apache2-igloo.allatou.dreamhost.com (67.205.11.189): icmp_seq=2 ttl=49 time=19.8 ms
    64 bytes from apache2-igloo.allatou.dreamhost.com (67.205.11.189): icmp_seq=3 ttl=49 time=20.2 ms
    64 bytes from apache2-igloo.allatou.dreamhost.com (67.205.11.189): icmp_seq=4 ttl=49 time=20.9 ms
    ^C
    --- prowse.tech ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 7ms
    rtt min/avg/max/mdev = 19.770/20.698/21.947/0.824 ms
    ```

    In the example, prowse.tech is being translated (or *resolved*) to the IP address 67.205.11.189. Also, we are receiving replies. This is what you want so that you can connect properly to the Internet.

> Note:
    For more information on Debian networking (for Debian servers with no GUI) then see this link:
    https://wiki.debian.org/NetworkConfiguration

## *systemd-networkd*

The systemd-networkd service (or simply networkd) is used by Ubuntu server and in some other special situations. For best results, use an Ubuntu server to accomplish the following labs.

### Lab 3-4
**Start and stop the networkd service**

- Check the service first:

    `systemctl status systemd-networkd`

- Stop the service

    `systemctl stop systemd-networkd`

- Start the service

    `systemctl start systemd-networkd`

### Lab 3-5
**Analyze and modify the netplan configuration**

- Open the netplan configuration:

    For example: `vim /etc/netplan/00-installer-config.yaml`

> Example:
    Here's an example of a netplan .yaml file in Ubuntu server:
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
  The network interface name is *enp1s0*. The IP address of the server is 10.0.2.53. The gateway and DNS server IPs are one and the same: 10.0.2.1. Though you don't have to, I am using square brackets for some of the IP addresses. By doing so, you can easily add IP addresses, by comma-separating them. For example: 
    `[10.0.2.53/24,10.0.2.158/24]` 

- Modify and apply the netplan configuration:

    If your configuration shows DHCP, then modify it to a static address, similar to the configuration above, but based on your IP network. Otherwise, change the IP address to something different. Then, save and quit out of vim.

    You can have netplan check your configuration by typing `netplan try`. 

    If there are no error messages, proceed by saving the configuration: `netplan apply` 

    Then test it with `ping example.com`

### Lab 3-6
**Analyze with networkctl**

- Use the `networkctl` command to find the available network interfaces. Here's an example of an Ubuntu Server's interfaces:

```
root@ubuntu-server:~# networkctl
IDX LINK   TYPE     OPERATIONAL SETUP     
  1 lo     loopback carrier     unmanaged 
  2 enp1s0 ether    routable    configured

2 links listed.
```

Here you can see that the system has the local loopback (like any Linux system by default), and the *enp1s0* Ethernet network interface. 

- Use the `networkctl status` command to see the status of networkd-based interfaces. Here's an example of the results of that command.

```
root@ubuntu-server:~# networkctl status
           State: routable                         
         Address: 10.0.2.53 on enp1s0              
                  fe80::5054:ff:febc:b314 on enp1s0
         Gateway: 10.0.2.1 on enp1s0               
             DNS: 10.0.2.1                         
  Search Domains: example.com                      

Mar 30 14:41:36 ubuntu-server systemd[1]: Starting Network Service...
Mar 30 14:41:36 ubuntu-server systemd-networkd[682]: Enumeration completed
Mar 30 14:41:36 ubuntu-server systemd[1]: Started Network Service.
Mar 30 14:41:36 ubuntu-server systemd-networkd[682]: enp1s0: IPv6 successfully enabled
Mar 30 14:41:36 ubuntu-server systemd-networkd[682]: enp1s0: Link UP
Mar 30 14:41:36 ubuntu-server systemd[1]: Starting Wait for Network to be Configured...
Mar 30 14:41:36 ubuntu-server systemd-networkd[682]: enp1s0: Gained carrier
Mar 30 14:41:37 ubuntu-server systemd-networkd[682]: enp1s0: Gained IPv6LL
Mar 30 14:41:37 ubuntu-server systemd[1]: Finished Wait for Network to be Configured.
```

As you can see, the state of the connection is "routable" which means that this system can connect out to other systems and beyond to other networks (if available). The command also shows the IP address of the *enp1s0* network interface, as well as the DNS and gateway IP addresses. The command also displays the status of the systemd-networkd service, which is started. 
 
> For more information, see the following links:
    
  Netplan Configurations: https://netplan.io/examples
     
  networkd on Debian: https://wiki.debian.org/SystemdNetworkd


## *NetworkManager*

The NetworkManager service is probably the most commonly used networking service in Linux. It is the default on RHEL/Fedora/CentOS, and is used by default on Debian (as a client), Ubuntu Desktop, and many more Linux distributions. 

### Lab 3-7
**Working with NetworkManager**

- View the status of the service:

	`systemctl status NetworkManager`

- Turn off the service:

	`systemctl stop NetworkManager`

- Test the network connection: 

	`ping example.com`

- Enable and start the service with one command:

	`systemctl --now enable NetworkManager`

- Test the network connection again to make sure it works. 

### Lab 3-8
**Analyze the different NetworkManager-related tools**

There are at least six ways to modify the networking configuration on Linux systems that use NetworkManager. Use a Fedora Workstation or CentOS Workstation to view each of these. (Most of them are also available on other distros such as Debian and Ubuntu clients.)

- **Settings** - Right-click the desktop and select Settings. Then, locate the Network option. Finally, click the gear in the Wired section. 

View all of the tabs on your system. In the figure we start with the Details tab. It shows that our IPv4 address is 10.0.2.55. You can make modifications to the IP settings in the IPv4 and IPv6 tabs. 

- **nm-connection-editor** - Open a terminal and simply type `nm-connection-editor`. This will display a graphical tool that looks quite similar to Settings>Network. 

- **nmtui** - Open a terminal and type `nmtui`. This is the NetworkManager tab-based user interface. It is a menu-driven system that is text only - it can run on a system with or without a desktop environment. 

- **nmcli** - Open a terminal and type `nmcli`. This will show you the networking configuration. The nmcli tool can also be used to modify the configuration either as single commands or within the nmcli shell. This is a popular tool because of its depth, and the fact that you can use it on servers and clients because it runs in the command line only. There will be a segment dedicated to nmcli later in the webinar.

- **Configuration files** - (Not recommended) Depending on the Linux distribution, the main networking configuration file(s) can be in different locations. 

	- RHEL/CentOS location: /etc/sysconfig/network-scripts
	- Many other distros: /etc/NetworkManager/system-connections

> Example on CentOS 8.2: 
	Here's an example of a network configuration file on a CentOS system. I accessed it by typing: 
	
	`vim /etc/sysconfig/network-scripts/ifcfg-ens3`

	```
	TYPE=Ethernet
	PROXY_METHOD=none
	BROWSER_ONLY=no
	BOOTPROTO=none
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=no
	IPV6INIT=yes
	IPV6_AUTOCONF=yes
	IPV6_DEFROUTE=yes
	IPV6_FAILURE_FATAL=no
	IPV6_ADDR_GEN_MODE=stable-privacy
	NAME=ens3
	UUID=539c56e8-ca22-4b5d-8ecd-cb244d1c24bd
	DEVICE=ens3
	ONBOOT=yes
	IPADDR=172.21.0.222
	PREFIX=16
	GATEWAY=172.21.0.1
	DNS1=172.21.0.1
	IPV6_PRIVACY=no
	ZONE=public
	```

You can see that all of the IP information is listed here. This particular system is on the 172.21.0.0 network and uses the IP address 172.21.0.222. The prefix is 16. That is another name for the netmask (or subnet mask). So the IP address could also be shown as 172.21.0.222/16. If for some reason the nmcli command doesn't work for you, the configuration file is a good backup plan!

---

> Example on a Debian Client:
	Here's an example of a network configuration file on a Debian client system (meaning one *with* a desktop environment). I accessed it by typing:

	`cd /etc/NetworkManager/system-connections`

	and opening the configuration file: `vim 'Wired connection 1.nmconnection`

	```
	[connection]
	id=Wired connection 1
	uuid=89321b95-d3b5-307d-a9f2-ff441c3f61ba
	type=ethernet
	autoconnect-priority=-999
	permissions=
	timestamp=1615917585

	[ethernet]
	mac-address=52:54:00:6D:FB:AC
	mac-address-blacklist=

	[ipv4]
	address1=10.0.2.52/24,10.0.2.1
	dns=10.0.2.1;
	dns-search=
	method=manual

	[ipv6]
	addr-gen-mode=stable-privacy
	dns-search=
	method=auto
	```

Here we find that the information is displayed a bit differently. Under the *ipv4* section we see the IP address of the system "10.0.2.52/24" followed by the gateway address "10.0.2.1", with the DNS server listed afterward. 

> Note:
	Changes to a network configuration file on a system that uses NetworkManager require either an `nmcli connection reload` or a system reboot to take effect. 

### Lab 3-9
**Working with the Cockpit Service**

Cockpit is a program that can be run on a server allowing admins to analyze and configure the server from a remote system via a web browser. It can be installed on most Linux distributions but is included in Fedora/RHEL/CentOS by default (though it might not be enabled and started). A Fedora server and Fedora workstation are recommended for this lab.

- **Enable and start the Cockpit service** - Type the following command to enable and start cockpit.
	`systemctl --now enable cockpit.socket`

- **Verify that it is running** 
	`systemctl status cockpit.socket`

- **Connect from a remote system** - Configure the server from a Linux workstation with a desktop environment. Open a browser and connect to https://ip_address:9090. Where *ip_address* is the IP address of your server that you enabled cockpit on. For example, type: 

	`https://10.0.2.54:9090`

	That should connect to the server. You may have to accept the security risk to continue, and then you will need to login to the server using the same credentials you have been using so far. After that, click "Networking" on the left-hand side. That wil display all of the networking information for the system.

### Lab 3-10
**Working with `journalctl`**

Linux keeps a "journal" or a list of events that have happened on the system. You can query this journal with the journalctl command. 

- Type `journalctl` and use the arrow keys or the pageup and pagedown keys to see some of the logs. There is a lot of information, so we need to filter it. Press 'q' to escape and continue on.

- Filter by the unit or service. Issue the following command:

	`journalctl -u NetworkManager`

	This displays the journal entries related to the NetworkManager service. -u allows us to select the service (or unit) that we need to learn more about. But there can still be a lot of information so we can use the tail option to show the last 10 lines.

- Using tail: Issue the following command to see the last 10 lines of the journal as they pertain to the NetworkManager service.
	
	`journalctl -u NetworkManager | tail -10`

- Modify the command further so that the information is sorted into columns and is outputted to less. 

	`journalctl -u NetworkManager | tail -10 | column -t | less`

	The results might look similar to this:

	```
	Mar  30  18:37:41  fed-server  NetworkManager[672]:  <info>  [1617154661.2648]  device    (enp7s0):  state        change:      config       ->          ip-config    (reason  'none',   sys-iface-state:  'managed')       
	Mar  30  18:37:41  fed-server  NetworkManager[672]:  <info>  [1617154661.2744]  device    (enp7s0):  state        change:      ip-config    ->          ip-check     (reason  'none',   sys-iface-state:  'managed')       
	Mar  30  18:37:41  fed-server  NetworkManager[672]:  <info>  [1617154661.2801]  device    (enp7s0):  state        change:      ip-check     ->          secondaries  (reason  'none',   sys-iface-state:  'managed')       
	Mar  30  18:37:41  fed-server  NetworkManager[672]:  <info>  [1617154661.2811]  device    (enp7s0):  state        change:      secondaries  ->          activated    (reason  'none',   sys-iface-state:  'managed')       
	Mar  30  18:37:41  fed-server  NetworkManager[672]:  <info>  [1617154661.2940]  device    (enp7s0):  Activation:  successful,  device       activated.                                                                     
	Mar  30  18:37:41  fed-server  NetworkManager[672]:  <info>  [1617154661.2963]  manager:  startup    complete                                                                                                              
	Mar  30  18:37:43  fed-server  NetworkManager[672]:  <info>  [1617154663.6130]  dhcp6     (enp7s0):  activation:  beginning    transaction  (timeout    in           45       seconds)                                     
	Mar  30  18:37:43  fed-server  NetworkManager[672]:  <info>  [1617154663.6143]  policy:   set        'Wired       connection   1'           (enp7s0)    as           default  for       IPv6              routing     and  DNS
	Mar  30  18:38:28  fed-server  NetworkManager[672]:  <warn>  [1617154708.4711]  dhcp6     (enp7s0):  request      timed        out                                                                                         
	Mar  30  18:38:28  fed-server  NetworkManager[672]:  <info>  [1617154708.4716]  dhcp6     (enp7s0):  state        changed      unknown      ->          timeout        
	```
 
	For more information on the journalctl command, type `journalctl --help`, and for more in-depth information, see the manual page: `man journalctl`

> Note: 
	For more information on NetworkManager, see the following links:

- NetworkManager documentation: 
	https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/getting_started_with_networkmanager
	
- NetworkManager on Debian: 
	https://wiki.debian.org/NetworkManager

## Summary
The three main networking services that we covered here are: networking, networkd, and NetworkManager. Each has its place and each has pros and cons compared to the others. 

You will normally find the networking service on a Debian server, the networkd service on an Ubuntu server, and the NetworkManager service on a Red Hat Enterprise Linux server (or Fedora/CentOS). 

The commands you use for each of the services will vary. The networking service will work well with commands such as `ip a`, `ifup`, `ifdown`, and modifying configuration files with VIM. networkd will work well with commands such as `ip a`, `networkctl`, and in the case of Ubuntu, `netplan`. The NetworkManager service is all about `nmcli` and the myriad of other tools available to you. 

Study the key points of these services and their respective Linux distributions and the configuration files and commands used within each.

---
