Here's the lab material for day 2 of the Linux Networking & Security Fundamentals webinar.

> Note: My main IP network is either: 10.0.2.0/24 or 10.0.42.0/16, depending on the lab being used for the day.

> I run a Debian server and client, Fedora server and client, Ubuntu server, and some mystery Linux distributions. (Really, it's whatever I have available at the time!)

> See [this link](https://prowse.tech/webinars) for more information on how to set up your lab to follow along.
	
---

# **Part I: Security 101**

This is the foundation of the day's concepts, but there is only one lab. The core concepts of this section are listed below. Descriptions for these concepts can be found in the [Security+ Cert Guide.](https://learning.oreilly.com/library/view/comptia-security-sy0-501/9780134781051/ch01.xhtml)

- Nothing is ever 100% secure
- Defense in Depth 
- Principle of Least Privilege
- Always be Watching

### Lab 1-1
**Using the dig command to check a website**

- Run the comand `dig -x 198.54.114.143` (or other IP address defined by the instructor). What do you find? 

- Ping that same IP address and view the results there as well.
> Note: If you do not have the `dig` command, you will need to install it. For example, in Debian-based systems: `sudo apt install dnsutils`.

This is actually the IP address of my website `dprocomputer.com` which resides on a shared server. 

