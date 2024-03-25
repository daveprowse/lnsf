# Part II: Working with Services

Services are the underlying programs that make applications work. For example, without the NetworkManager service, we wouldn't be able to use the Apache web server. Services are important for two reasons:
 
1. They make the system *available* to legitimate users.
2. They increase the attack surface, and in so doing, make the system available to illegitmate users too.

So, we have to split our brain. One side needs to think about availability and the other needs to consider security. This is the constant balance that we must maintain in the tech world.

### Lab 2-1
**Reducing the Attack Surface**

This lab shows how to stop and disable a service, and check the open ports of a system. The lab is run on a Debian client system.

- Stop the networking service

	`systemctl stop networking`

	Then check its status with `systemctl status networking`

- Disable the service 

	`systemctl disable networking`

	Then check its status again.

	> Note: The service can be stopped *and* disabled with one command:
		
		`systemctl --now disable networking`

- Start and enable the networking service

	`systemctl --now enable networking`

- Check for open ports being used by services

	`ss -tulnw`

	Example results:

	```
	user@deb52:~$ ss -tulnw
	Netid         State           Recv-Q          Send-Q                    Local Address:Port                    Peer Address:Port         
	icmp6         UNCONN          0               0                                     *:58                                 *:*            
	udp           UNCONN          0               0                         192.168.122.1:53                           0.0.0.0:*            
	udp           UNCONN          0               0                        0.0.0.0%virbr0:67                           0.0.0.0:*            
	udp           UNCONN          0               0                               0.0.0.0:44050                        0.0.0.0:*            
	udp           UNCONN          8448            0                         192.168.122.1:5353                         0.0.0.0:*            
	udp           UNCONN          8448            0                             10.0.2.52:5353                         0.0.0.0:*            
	udp           UNCONN          0               0                               0.0.0.0:5353                         0.0.0.0:*            
	udp           UNCONN          0               0                               0.0.0.0:5353                         0.0.0.0:*            
	udp           UNCONN          0               0                               0.0.0.0:56590                        0.0.0.0:*            
	udp           UNCONN          0               0                                  [::]:45721                           [::]:*            
	udp           UNCONN          0               0                                  [::]:5353                            [::]:*            
	tcp           LISTEN          0               100                           127.0.0.1:20959                        0.0.0.0:*            
	tcp           LISTEN          0               100                           127.0.0.1:20000                        0.0.0.0:*            
	tcp           LISTEN          0               100                             0.0.0.0:4000                         0.0.0.0:*            
	tcp           LISTEN          0               100                           127.0.0.1:12001                        0.0.0.0:*            
	tcp           LISTEN          0               100                           127.0.0.1:12002                        0.0.0.0:*            
	tcp           LISTEN          0               100                           127.0.0.1:25001                        0.0.0.0:*            
	tcp           LISTEN          0               80                            127.0.0.1:3306                         0.0.0.0:*            
	tcp           LISTEN          0               100                           127.0.0.1:26002                        0.0.0.0:*            
	tcp           LISTEN          0               32                        192.168.122.1:53                           0.0.0.0:*            
	tcp           LISTEN          0               9                               0.0.0.0:21                           0.0.0.0:*            
	tcp           LISTEN          0               128                             0.0.0.0:22                           0.0.0.0:*            
	tcp           LISTEN          0               128                           127.0.0.1:7001                         0.0.0.0:*            
	tcp           LISTEN          0               100                                [::]:4000                            [::]:*            
	tcp           LISTEN          0               128                                   *:80                                 *:*            
	tcp           LISTEN          0               9                                  [::]:21                              [::]:*            
	tcp           LISTEN          0               128                                [::]:22                              [::]:*            
	tcp           LISTEN          0               128                               [::1]:7001                            [::]:*    
	```

	Scroll over to see the open ports. You can see that the DHCP client (port 67), DNS (port 53), SSH (port 22), and several others are opened. Those are ports that aare opened on the inbound side. Every open port is a potential vulnerability. Normally, a client such as this would have no legitimate reason to have port 22 open (amoong other ports). We could close that port by stopping and disabling the service with one command. 

	Don't do this on your system (so you don't lose SSH) but here is an example:

	`systemctl --now disable ssh`

	You can use curly braces {} to stop and disable multiple services at the same time. For example:

	`systemctl --now disable {service1,service2}`

### Lab 2-2
**Cause a Degraded System - and fix it!**

This lab demonstrates how one failed service or program will cause a "degraded system". The lab is run on a Debian client.

- Cause a service to fail.

	Here we'll type an error into the SSH server configuration file. When we attempt to restart SSH, it will fail. That will then cause the system to become degraded.  

	1. Open the sshd_config file. It is located in /etc/ssh.
	2. Locate the line that says `#Port 22`, and modify it to read `Port22----$%^`
	3. Restart the SSH server service: `systemctl restart sshd`. You should see the failure. For example:


	```
	root@deb52:/etc/ssh# systemctl restart sshd
	Job for ssh.service failed because the control process exited with error code.
	See "systemctl status ssh.service" and "journalctl -xe" for details.
	```

- Start troubleshooting

	Now, run the command `systemctl status` and the second line should read "State: degraded". Unfortunately, this command does not show what has failed, or how. But you can find any failed services by issuing the command: `systemctl --failed`. This should result in the following:

	```
	root@deb52:/etc/ssh# systemctl --failed
	  UNIT        LOAD   ACTIVE SUB    DESCRIPTION                
	● ssh.service loaded failed failed OpenBSD Secure Shell server

	LOAD   = Reflects whether the unit definition was properly loaded.
	ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
	SUB    = The low-level unit activation state, values depend on unit type.

	1 loaded units listed. Pass --all to see loaded but inactive units, too.
	To show all installed unit files use 'systemctl list-unit-files'.
	```

	We can see that the ssh service has failed. We can check the ssh and sshd servicese individually with the systemctl or journalctl commands. Examples:

	`systemctl status ssh` and `systemctl status sshd`

	`journalctl -u ssh` and `journalctl -u sshd`

	If systemctl doesn't go back in time far enough in the logs, then journalctl is the tool to use. By issuing the command `journalctl -u ssh` we can see why and where the actual error occurred:

	```
	Apr 07 16:48:15 deb52 sshd[5193]: /etc/ssh/sshd_config line 13: Badly formatted port number.
	```

	It tells us exactly where to look to analyze the problem more: the sshd_config file on line 13. That is, of course, what we messed up in the first place! 

	> Note: You could have also run the `journalctl -xe` command (as the Linux error message stated) but that will give you a ton of information that you would need to sift through. Using the -u option with the particular service name will usually save you time. And you can put that extra time in a bottle for later use if you wish.

- Fix the problem and verify full functionality

	Access the SSH configuration file:

	`vim /etc/ssh-sshd_config`

	Change the line that we modified earlier back to the original: `#Port 22`

	Restart the service:

	`systemctl restart sshd`

	Check the system with a `systemctl status`. At this point, it should once again say that the State is "running" in green. And all is well in Linux Land. 