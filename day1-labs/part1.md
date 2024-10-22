# Part I: Networking 101

In Part I of the webinar we start with some basic computer networking including a review of a typical local area network setup, basic TCP/IP, and a primer of the OSI model. This portion of the webinar is designed to act as a foundation for the rest of the course.

## *TCP/IP*

TCP/IP is a suite of protocols used by computers to communicate with each other. It includes the protocols TCP, and IP, but also many others, such as ICMP, ARP, HTTP, FTP, POP, SMTP, DNS, DHCP, and so on. IP addresses are used by each computer that runs TCP/IP to identify them, and to facilitate the communication of packets of data. 

A typical example of an IP network is: 192.168.1.0. The network portion of the address is 192.168.1. The host portion is 0. It's the host portion that differentiates between systems on the same network. For example, one computer might have the IP address 192.168.1.73. Another might have 192.168.1.142. And the gateway might use 192.168.1.1. It's that last octet, that last number that sets each system apart. Other IP networks can have a larger host portion. For example, in the 172.17.0.0 network, the network portion is 172.17, and the host portion is 0.0. So, a system on the 172.17 network might have an address such as 172.17.51.3. "51.3" would be the host portion of the IP address. 

### Lab 1-1

**Use the `ip a` command**

- In the console (or terminal) type the `ip a` command and view the results. Here's an example on a Debian server:

```
root@deb51:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:f0:2b:b4 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.51/24 brd 10.0.2.255 scope global enp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fef0:2bb4/64 scope link 
       valid_lft forever preferred_lft forever
3: enp7s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 52:54:00:0a:2a:c9 brd ff:ff:ff:ff:ff:ff
```

This shows two interfaces: 

1. *lo*, which is the local loopback, something that is found by default on every system that runs TCP/IP. "inet" shows the IPv4 address, which for *lo* is 127.0.0.1.
2. *enp1s0*, which is the main network interface, used to access other systems and the Internet. The IPv4 address is 10.0.2.51.

> Note:
	For help with the ip command type `ip --help`, and for more in-depth information, see the manual page `man ip`. 

## *The OSI Reference Model*

The Open Systems Interconnection (OSI) reference model is used to define how data is transmitted and received between systems. It defines how protocols work, and how programs will use those protocols. It is made up of seven layers, listed from layer 7 down to layer 1 below:

- Layer 7:  Application
- Layer 6:  Presentation
- Layer 5:  Session
- Layer 4:  Transport
- Layer 3:  Network
- Layer 2:  Data Link
- Layer 1:  Physical

Different protocols work on different layers. For example, in Lab 1-1 we used the `ip a` command. This displayed the IP address of the system (10.0.2.51). IP addresses exist on layer 3 of the OSI model: the network layer. Other protocols work on other layers. For instance, TCP works on layer 4 (transport), and HTTP works on layer 7 (application). When you have different protocols working on different layers, they are considered to be *stacked* upon each other. This leads to terms such as "OSI stack", "TCP/IP stack", or simply "network stack". The OSI model can be very helpful when designing programs, designing networks, and troubleshooting network connections, as well as doing packet analysis. 

 
There are no other labs in this section. See the reference links below for more information about TCP/IP and the OSI model.

### TCP/IP Reference Links

- TCP/IP Guide (2005): <https://learning.oreilly.com/library/view/tcpip-guide/9781593270476/>
- TCP/IP Illustrated (2011): <https://learning.oreilly.com/library/view/tcpip-illustrated-volume/9780132808200/>
- Computer Networks & Internets (Douglas Comer) - if you can get your hands on one...

### OSI Reference Model links

https://en.wikipedia.org/wiki/OSI_model

https://www.youtube.com/watch?v=m_RfrAfUFx8

---
