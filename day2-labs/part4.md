# Part IV: Firewalls

A firewall is essentially a set of rules that can permit or deny traffic to a particular location. they can be hardware-based or software-based. Another commonly used name for a firewall is a packet filter. 

In the following two labs we will work on UFW and firewalld.

## ***UFW***

UFW is the Uncomplicated FireWall. It was developed by Ubuntu and works on Ubuntu and Debian-based systems. We'll run the UFW lab on a Debian server.

### Lab 4-1
**Working with UFW**

The Uncomplicated FireWall (developed by Ubuntu), is designed for Ubuntu and works well on Debian. It is not designed for Red Hat and its derivatives.
This is a front-end firewall utility that interfaces with the iptables back-end tool and ultimately, the iptables (or nftables) themselves. 

In this lab we'll show how to install UFW, enable the UFW service, enable the UFW firewall itself, and create and remove some rules - testing as we go. I'll be testing on a Debian server, but an Ubuntu server would work just as well.

> Warning! You will need to work locally at the virtual machine for this lab. This is because once enabled, the firewall will shutdown subsequent SSH connections. 

- Setup UFW

	To begin, we'll install and enable UFW. On Ubuntu it should already be installed. However, other distributions, such as Debian, will need to install it. 

	`apt install ufw`

	After that we need to make sure the service is enabled and started.

	`systemctl status ufw`

	If it is not enabled and started, do so:

	`systemctl --now enable ufw`

	Now, turn on the firewall itself.

	`ufw enable`

	That's it, UFW is enabled and running. Let's take a look at the main configuration file for UFW:

	`vim /etc/default/ufw`

	Note the default global rules. All inbound and forwarded connections are being dropped, while any outbound connections are allowed. Here's an example of that on the Debian system:

	```
	# Set the default input policy to ACCEPT, DROP, or REJECT. Please note that if
	# you change this you will most likely want to adjust your rules.
	DEFAULT_INPUT_POLICY="DROP"

	# Set the default output policy to ACCEPT, DROP, or REJECT. Please note that if
	# you change this you will most likely want to adjust your rules.
	DEFAULT_OUTPUT_POLICY="ACCEPT"

	# Set the default forward policy to ACCEPT, DROP or REJECT.  Please note that
	# if you change this you will most likely want to adjust your rules
	DEFAULT_FORWARD_POLICY="DROP"
	```

	> Note:
		For the actual rules configuration file go to /etc/ufw/ and access user.rules to start. 

- Create inbound rules

	Now that we enabled the firewall, we can't connect to the server anymore! If you were to test it from a client, SSH connections would fail. But most of the time, you will want to run a firewall - but with exceptions. One exception is SSH. Let's enable inbound ssh with the following command:

	`ufw allow 22/tcp`

	or

	`ufw allow ssh`

	Now, lets show the rule:

	`ufw show added`

	This should show that the UFW rule has been added to allow inbound SSH. Now, attempt to SSH into the server from a client system. It should work.

	> Note:	To view the access log, go to /var/log/ufw.log

	In some firewalling scenarios you might need to deny a service. Let's deny SSH now with the following command:

	`ufw deny ssh`

	Enter the `ufw status` command to verify that it is working. You will note that the "SSH allow" rule is gone. The deny rule now takes precedence as shown below:

	```
	root@deb51:~# ufw status
	Status: active

	To                         Action      From
	--                         ------      ----
	22/tcp                     DENY        Anywhere                  
	22/tcp (v6)                DENY        Anywhere (v6)   
	```

- Delete a rule (and more!)

	It could be that you want to allow SSH over IPv4, but not over IPv6. UFW creates separate rules for both IP systems (by default) unless you specify otherwise. In this case, we have both set to deny. We could delete the IPv4, and afterward create a rule that allows IPv4 SSH access. 
	
	But first, we delete the SSH IP4 rule. The easiest way to do this is to delete the rule *number*. to find out the rule numbers, issue the following command:

	`ufw status numbered`

	You should see the rule numbers on the left, for example: 

	```
	root@deb51:~# ufw status numbered
	Status: active

	     To                         Action      From
	     --                         ------      ----
	[ 1] 22/tcp                     DENY IN     Anywhere                  
	[ 2] 22/tcp (v6)                DENY IN     Anywhere (v6)   
	```
	
	So the rule we wnat to delete is Rule #1. Do this with the following command:

	`ufw delete 1`

	That's it. Check it again with `ufw status` and it should be gone. Now create an allow rule for SSH over IPv4 only. And check it:

	```
	root@deb51:~# ufw allow proto tcp to 0.0.0.0/0 port 22
	Rule added
	root@deb51:~# ufw status numbered
	Status: active

	     To                         Action      From
	     --                         ------      ----
	[ 1] 22/tcp                     ALLOW IN    Anywhere                  
	[ 2] 22/tcp (v6)                DENY IN     Anywhere (v6)    
	```
	
	Now we can see that SSH over IPv4 is allowed, but SSH over IPv6 is not. Great work!

	> Note:	That was just one example of many. And the networking options are vast. You will need to tweak your rules based on your network configuration and needs.

	> Note:
		If you find that you don't need IPv6, you can simply disable it globally. Go to:

		/etc/default/ufw

		and change IPV6=YES to IPV6=NO. Then reload the service: `ufw reload`. Alternatively, you could restart the UFW service: `systemctl restart ufw` or `ufw disable` and `ufw enable`.

- Disable UFW, delete the rules, and disable the UFW service

	Careful with the following command. It removes the rules and resets the firewall to a disabled state!

	`ufw reset`

	Then, stop and disable the service to bring the system back to its original state before we began the lab.

	`systemctl --now disable ufw`

> Note:
	For more information about UFW:

	Help file: `ufw --help`

	MAN page: `man ufw`

	Basic usage: <https://help.ubuntu.com/community/UFW>{target="_blank}
	More advanced usage and configs: <https://wiki.ubuntu.com/UncomplicatedFirewall>{target="_blank"}

## ***firewalld***

firewalld is a front-end firewalling utility developed by Red Hat. So it's designed for Fedora, RHEL, CentOS, and other Red Hat derivatives, and is installed on them by default.  It acts as an easier to use front-end for the netfilter framework via the nftables userspace utility - which essentially is the *nft* command. 

> Note:	This utility is recommended for workstations. While it is often used on servers, it is recommended to use nftables (and the nft utility) on servers and for firewalling entire networks. 

firewalld can be installed on Debian and Ubuntu, but it is not designed for them and therefore might not work quite as expected. 

For the following lab we'll work with a Fedora server. 

### Lab 4-2
**Working with firewalld**

> Warning! You will need to work locally at the virtual machine for this lab. This is because once enabled, the firewall will shutdown subsequent SSH connections. 

- Check if the service is running:

	`systemctl status firewalld`

	If not started and enabled, do it now:

	`systemctl --now enable firewalld`

- Show the current zone

	Zones are entities that hold different configurations. For example, *block* and *deny* are more secure built-in zones, whereas *public* is a bit less secure (and commonly found as a default). 

	Almost all firewalld commands start with `firewall-cmd`. To show the current zone, issue the following command:

	`firewall-cmd --get-active-zones`

	Here's an example of the output on the Fedora server:

	```
	[root@fed-server ~]# firewall-cmd --get-active-zones
	FedoraServer
	  interfaces: enp1s0
	```

	Here we can see that the current active zone that the firewall is using is called *FedoraServer*. This is the default option, and is fairly limited.  It is very similar to the *public* zone (which is used by CentOS for example by default). 

	We also see that the network interface (enp1s0) is making use of that zone type. 

	You can also issue the following command to get more information about the active zone:

	`firewall-cmd --list-all`

	Here is the results from our Fedora Server:

	```
	[root@fed-server ~]# firewall-cmd --list-all
	FedoraServer (active)
	  target: default
	  icmp-block-inversion: no
	  interfaces: enp1s0
	  sources: 
	  services: cockpit dhcpv6-client ssh
	  ports: 
	  protocols: 
	  masquerade: no
	  forward-ports: 
	  source-ports: 
	  icmp-blocks: 
	  rich rules: 
	```

	You can see the services that are running, and that is essentially what we are limited to. So we can ping the server, SSH into it, use the cockpit service to control it remotely from a web browser, and the server can seek to obtain an IP address from a DHCP server. We can't do much of anything else though. However, even this will not be secure enough for many environments. 

	**Example "You try it!"**
		
    Try now to ping and SSH into the Fedora server. You should be able to.

	> Note:
		There are many types of built-in zones. To see a list of them, type the following command:

		`firewall-cmd --get-zones`

		To see in-depth information about all zones, type the following:

		`firewall-cmd --list-all-zones`

		You can also create your own zones if you wish.

- Change the active zone from *FedoraServer* to *block*

	Let's make the system more secure by changing the default active zone. Issue the following command:

	`firewall-cmd --zone=block --change-interface=enp1s0 --permanent`

	!!! note
		You will have to change the name of your interface from *enp1s0* to whatever your network interface name is. If you are not sure than issue either the `ip a` command or the `nmcli` command. Be sure to use the *Linux name* of the network interface, and not the NetworkManager name.

	This command changes the active zone. Check it with:

	`firewall-cmd --get-active-zones`

	Your interface should now be part of the *block* zone, making it more secure. Once again, you can list more information, but this time we have to add the zone name. Use the following command:

	`firewall-cmd --zone=block --list-all`

	You should see output similar to the following:

	```
	[root@fed-server ~]# firewall-cmd --zone=block --list-all
	block (active)
	  target: %%REJECT%%
	  icmp-block-inversion: no
	  interfaces: enp1s0
	  sources: 
	  services: 
	  ports: 
	  protocols: 
	  masquerade: no
	  forward-ports: 
	  source-ports: 
	  icmp-blocks: 
	  rich rules: 
	```

	Note that there are no services listed. So this zone type is more secure because it allows nothing to come through, including ping or SSH which we could accmplish earlier. You can view this further by typing the following command:

	`firewall-cmd --zone=block --list-ports`

	That command should show nothing because no ports are open right now, which in turn is because no services are currently allowed. 

	**Example "You try it!"**
		
    Go ahead and attempt to ping the Fedora server from a remote system and SSH into it. You should not be able to do so now. 


	Now try scanning the server with nmap. If you don't have nmap installed, you can easily install it (`apt install nmap` or dnf install nmap`, etc...)	

	Here I scan the Fedora server (10.0.2.54) from a Fedora client. As you can see below, the results show that everything is filtered, and effectively no ports are open.

	```
	[root@fed-client ~]# nmap -Pn 10.0.2.54
	Starting Nmap 7.80 ( https://nmap.org ) at 2021-04-09 13:51 EDT
	Nmap scan report for 10.0.2.54
	Host is up (0.00039s latency).
	All 1000 scanned ports on 10.0.2.54 are filtered
	MAC Address: 52:54:00:B5:29:89 (QEMU virtual NIC)

	Nmap done: 1 IP address (1 host up) scanned in 5.37 seconds
	```

- Add a rule to allow SSH

	Most of the time, we desire access to a server via SSH. So let's add a rule that allows it.

	`firewall-cmd --zone=block --add-port=22/tcp --perm`

	We have to specify the zone we are dealing with (block). The --add-port option has the ability to be configured by port number (22/tcp) or by name (ssh). We also added the --permanent option in abbreviated form (--perm) so that the port will remain open if the system is rebooted. 

	Now we have to reload the firewall for the change to take effect.

	`firewall-cmd --reload`

	Now we can check our work with the following commands:

	`firewall-cmd --zone=block --list-ports`

	`firewall-cmd --zone=block --list-all`

	Here are the results of the second option:

	```
	[root@fed-server ~]# firewall-cmd --zone=block --list-all
	block (active)
	  target: %%REJECT%%
	  icmp-block-inversion: no
	  interfaces: enp1s0
	  sources: 
	  services: 
	  ports: 22/tcp
	  protocols: 
	  masquerade: no
	  forward-ports: 
	  source-ports: 
	  icmp-blocks: 
	  rich rules: 
	```

	You can see that for the "block" zone, the only port that is listed is port 22. If we had run the command with the `--add-port=ssh` option, that would show up under services. Whether you do this by service name or by port is up to you but it might be dictated by organizational policy as well. Historically I prefer to do it by port number. 

	**Example "You try it!"**
		
    Now you should be able to SSH into the server, but not *ping* the server. 

	> Note:
		A subsequent nmap scan from the client: `nmap -Pn 10.0.2.54` should show that everything is filtered except for the SSH port. 

- Remove the SSH rule
	
	To remove the SSH rule, issue the following command:

	`firewall-cmd --zone=block --remove-port=22/tcp --perm`

	Then, reload the firewall:

	`firewall-cmd --reload`

	Check your work:

	`firewall-cmd --zone=block --list-ports`

	It should show that no ports are opened once again. 

- Change the active zone back to the original.

	Issue the following command:
	
	`firewall-cmd --zone=FedoraServer --change-interface=enp1s0 --per`

	> Note:
		Your original zone name might be different. For example, if you are working on CentOS, it is probably named *public*. 

		Your interface name may be different as well.

		Also, we have abbreviated --permanent even further to --per.

	Verify the active zone:

	`firewall-cmd --get-active-zones`

- Open multiple ports at once

	Now we have our original active zone. But it doesn't allow for things like HTTP, DNS, and so on. If we want those, we have to create a rule to open those ports. For example, if we wanted to run a FreeIPA server, we would need to open several ports. We could do it in the following manner:

	```
	firewall-cmd --add-port={80,443,389,636,88,464}/tcp --per
	```

	That will open up the ports associated with HTTP, HTTPS, LDAP, Secure LDAP, and two Kerberos-based ports. 

	Reload the firewall and the check your work:

	`firewall-cmd --reload`

	`firewall-cmd --list-ports`

- Close multiple ports at once

	```
	firewall-cmd --add-port={80,443,389,636,88,464}/tcp --per
	```

	Reload the firewall, and check your work. Everything should be back to the original firewall settings.

- Stop the firewall service and scan the system

	Stop and disable the firewall service with the following command:

	`systemctl --now disable firewalld`

	Now scan it from a remote client:

	`nmap 10.0.2.54`

	Change the IP address as necessary for your server. Here are example results:

	```
	[root@fed-client ~]# nmap 10.0.2.54
	Starting Nmap 7.80 ( https://nmap.org ) at 2021-04-09 14:19 EDT
	Nmap scan report for 10.0.2.54
	Host is up (0.00015s latency).
	Not shown: 998 closed ports
	PORT     STATE SERVICE
	22/tcp   open  ssh
	9090/tcp open  zeus-admin
	MAC Address: 52:54:00:B5:29:89 (QEMU virtual NIC)

	Nmap done: 1 IP address (1 host up) scanned in 0.24 seconds
	```

	We didn't need the -Pn option this time because the Fedora server is not firewalled at all. You'll also note that the scan only took 0.24 seconds. It shows us that the SSH and zeus-admin (cockpit) ports are open. When you do this type of work, you get to see why firewalls are so important. 

	> Note:	There is a graphical version of the firewall-cmd command structure. It is called *firewall-config*, and it can be installed to a Fedora client with the `dnf install firewall-config` command. 

	**Success!**
		
    That's it for this lab. Great work!