# Part VI: Securing SSH

SSH is a heavily used tool. However, it is vulnerable (like everything else). This section describes how to secure an SSH server.

Remember that a typical SSH connection is performed from one computer to another. For example, if *computerA* wanted to SSH into *computerB* then the user at computerA would type:

`ssh user@computerB`

If the user at computerA knows the password of the user account at computerb, then access will be granted. However, while passwords offer some security, it's more secure to use cryptographic keys. 

### Lab 6-1
**Creating and Distributing SSH Keys**

In this lab we'll build an SSH key on a Debian client (as a typical user), distribute it to a Debian server, and then SSH into that server from the client using the key we just made.

To SSH into a system using a key, you actually need to create a key *pair*. The first key is called a private key and it resides at the computer where the key pair is built. The second key is called a public key, and it resides at the SSH server, meaning the remote computer that will be connected to. 

When you attempt to SSH into the remote system, that system presents its public key. If it matches the private key stored at the local computer, then the connection is allowed. If it doesn't match (for example, a rogue or malicious system) then the connection will be denied. 

- Create an RSA-based key pair
https://halftimebeverage.com/warrior-21-68865
	To create an SSH key pair, login to the client as a typical user, and type the following:

	`ssh-keygen`

	When it prompts you, press enter to save the key pair to the default location. It's also highly recommended to select and confirm a complex passphrase to protect the keypair.  The process will be very similar to the following:

	```
	user@deb52:~$ ssh-keygen
	Generating public/private rsa key pair.
	Enter file in which to save the key (/home/user/.ssh/id_rsa): 
	Enter passphrase (empty for no passphrase): 
	Enter same passphrase again: 
	Your identification has been saved in /home/user/.ssh/id_rsa.
	Your public key has been saved in /home/user/.ssh/id_rsa.pub.
	The key fingerprint is:
	SHA256:amG7G71T2CxDiruTOhO+1niqtV7X6Zer9B3BGjpWKTQ user@deb52
	The key's randomart image is:
	+---[RSA 2048]----+
	|                 |
	|                 |
	|        E        |
	|       ... o     |
	|     .ooS++ o    |
	|  . ...*+=+o .   |
	| ..+.o* O+...    |
	| .*o*o *.+o. .   |
	|.+*B.oo.++o..    |
	+----[SHA256]-----+
	user@deb52:~$ 
	```

	**Danger** 
  
  Pressing enter for a blank passphrase is insecure. For added security, use a passphrase. The more length and complexity the better. This protects the key pair that you have created. Think about it: If someone was to gain access to your computer (internally or externally), and more importantly to your user account, then that person could potentially gain access to all the systems that you would normally access via SSH. 

	> Note:	When you do use a passphrase fo the SSH key pair, you will be required to type it the first time you connect to a remote host using the SSH key. Subsequent attempts during a single login session will not require it unless you are doing double SSH: meaning if you are SSH'd into the client, and then are SSH'ing from there into another system. However, all of this is configurable at the system where the keys were created in the /etc/ssh/ssh_config file. Just be careful of how loose you are with security!

- View the SSH key pair

	The key pair is saved in a hidden directory. Type `ls -a` to see it. The directory name is .ssh. Access that directory: `cd .ssh`. Then type `ls` to view the contents. The procedure should be similar to the following:

	```
	user@deb52:~$ ls -a
	.              .bash_logout           .cache   Documents         .gnupg         .local    Music           Pictures  .qt             .ssh       .viminfo
	..             .bash-powerline-ng.sh  .config  Downloads         .ICEauthority  .mono     .nmcli-history  .profile  .recently-used  Templates  .wget-hsts
	.bash_history  .bashrc                Desktop  .fancy-prompt.sh  .lesshst       .mozilla  .nx             Public    script.sh       Videos     .Xauthority
	user@deb52:~$ cd .ssh
	user@deb52:~/.ssh$ ls
	id_rsa  id_rsa.pub  known_hosts
	user@deb52:~/.ssh$ 
	```

	You can see three files inside of .ssh: id_rsa (which is the private key), id_rsa.pub (which is the public key to be copied to remote hosts), and known_hosts, which contains the list of known hosts that you have connected to in the past. 

	!!! tip
		You can make bigger keys if your organization requires it. A typical RSA-based SSH key is 2048-bit. Let's say you needed 3072-bit. You could type the following:
	
		`ssh-keygen -b 3072`

		Remember to use different names/directories when creating subsequent keys. 

- Copy the public key to a remote host

	Now that our key pair is created, we can copy the public key of our key pair to a remote host. In this lab the key will be copied to a Debian server, making use of the user account that exists there. 

	`ssh-copy-id user@10.0.2.51`

	You will be required to type the password of the user account at the remote computer, but otherwise, that's it. You don't have to specify a directory, it will copy it to the default .ssh directory in the user accounts home directory. Just make sure that the results show that 1 key was added. Example:

	```
	user@deb52:~/.ssh$ ssh-copy-id user@10.0.2.51
	/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/id_rsa.pub"
	/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
	/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
	user@10.0.2.51's password: 

	Number of key(s) added: 1
	```

	Now try logging into the machine, with:   "ssh 'user@10.0.2.51'"
	and check to make sure that only the key(s) you wanted were added.

- SSH into the remote host

	Now, we can SSH into the remote host to take control of it. 

	`ssh user@10.0.2.51`

	The command is the same as normal. For the first time, the local system will require that we enter the SSH key passphrase (if we set one). Example:

	```
	user@deb52:~$ ssh user@10.0.2.51
	Enter passphrase for key '/home/user/.ssh/id_rsa': 
	Linux deb51 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

	The programs included with the Debian GNU/Linux system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.

	Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
	permitted by applicable law.
	Last login: Mon Apr 12 17:49:33 2021 from 10.0.2.52
	user@deb51:~$ 
	```

	As you can see, we are using a system *deb52* and SSH'd into a remote server *deb51*. 

	That's it. Now we are logged in using SSH keys. This way, we don't have to send a user's password over the network, and instead rely on the much more secure SSH key process. 

### Lab 6-2
**Securing the SSH Server**

Using SSH by itself is much more secure than older alternatives such as Telnet. But SSH has its own set of vulnerabilities. This section demonstrates how to make SSH more secure than it already is. In this lab we'll increase the SSH security of a Debian server in several ways. The bulk of these are configurations that are done in the /etc/ssh/sshd_config file.

1. Change the SSH inbound port

	An SSH server uses port 22 inbound by default. This is well-known. However, many port scans that are peformed by would be invaders only scan for the first 1000 ports of a system. If we change the port to something less known that is above the first 1000 ports, then it can increase the security of the server.
	
	On the Debian server, open the sshd_config file:

	`vim /etc/ssh/sshd_config`

	Find the line that indicates the port number. It should be near the beginning of the file. By default it is commented out, but it will say `#Port 22`. 

	As an example, we'll change it to port 2222. So change the line that reads `#Port 22` to:
	
	`Port 2222`

	Here's an example of the first 15 lines of the file:

	```
	#	$OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

	# This is the sshd server system-wide configuration file.  See
	# sshd_config(5) for more information.

	# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

	# The strategy used for options in the default sshd_config shipped with
	# OpenSSH is to specify options with their default value where
	# possible, but leave them commented.  Uncommented options override the
	# default value.

	Port 2222
	#AddressFamily any
	#ListenAddress 0.0.0.0
	#ListenAddress ::
	```

	Save and close the file.

	Then, restart the sshd service:

	`systemctl restart sshd`

	Now, on the Debian client, attempt to ssh into the server using the new port:

	`ssh user@10.0.2.51 -p 2222`

	The -p option specifies the port to use. You should see results similar to the following:

	```
	user@deb52:~$ ssh user@10.0.2.51 -p 2222
	Enter passphrase for key '/home/user/.ssh/id_rsa': 
	Linux deb51 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

	The programs included with the Debian GNU/Linux system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.

	Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
	permitted by applicable law.
	Last login: Mon Apr 12 17:52:58 2021 from 10.0.2.52
	user@deb51:~$ 
	```

	After successfully connecting, log back out. 

2. Disable password-based SSH

	Although our client will now connect to the server with a key instead of a password, the server can still be accessed by other systems and other users via password. We can disable password-based SSH altogether on the server. 

	As root on the Debian server, open /etc/ssh/sshd_config again and find the line that says:

	`#PasswordAuthentication yes`

	and uncomment it and change it to:

	`PasswordAuthentication no`

	Save and exit the file. This will disallow password-based authentication to that SSH server across the board. 

3. Restrict access by creating an exclusive group

	Don't just allow anyone to connect! By creating an exclusive group, we specify and organize the users that are allowed to connect.

	Create a new group. For example:

	`addgroup ssh-allowed`

	Now, add the user account to the new group:

	`adduser user ssh-allowed`

	Verify the existence of "user" within the "ssh-allowed" group:

	`groups user`

	Open the SSH Server config file once again:

	`vim /etc/ssh/sshd_config`

	and add the group to the end of the config file:

	`AllowGroups ssh-allowed`

	> Tip:
		You could also echo that line to the file without opening it:

		`echo "AllowGroups ssh-allowed" >> /etc/ssh/sshd_config`
    
    Just be sure to append the file with a double >> . This applies to any lines that we add to config files, but for these labs I want you to open and get to know the configuration files. 

	Now, test the SSH connection to the server with the user account. It should work.

	> Warning! Don't forget that we changed the port to 2222!

	Next, remove the user acount from the ssh-allowed group.

	`deluser user ssh-allowed`

	Now attempt to SSH in again. It should fail, as all other accounts that are not members of "ssh-allowed" will fail. 

4. Disable root login altogether

	While root cannot login to an openSSH server via password by default, the root account can still connect via a key. We take that ability away by modifying the following line in the sshd_config file:

	`PermitRootLogin prohibit-password`

	to:

	`PermitRootLogin no`

5. Lower the maximum amount of authentication attempts

	When a user connects to an SSH server with a password, the user get's 5 attempts to type the correct password. (The actual number listed is 6.) We can lower thisnumber to 4, which effectively allows the user 3 attempts. So, three strikes and you're out!

	Find the line: `MaxAuthTries` and change it from 6 to 4. 

There is plenty more that you can do. Some things to consider include:

- Lowering the login grace time
- Configure SSH timeouts (so that sessions don't last forever)
- Proper key management
- Use Fail2Ban to track and ban SSH connections

And much more. 

> Info:	For more information about SSH guidelines, see this NIST document:

https://nvlpubs.nist.gov/nistpubs/ir/2015/nist.ir.7966.pdf

**Summary**

Think of how many hackers attempt to get into servers that allow inbound SSH connections. There are literally millions, not to mention bots. So... you could argue that there can't be enough protection for an SSH server. 

So some fundamental ways to increase the security of the SSH server are to: use a different and less known port; disable password-based SSH; restrict SSH access by user and group; disable root login; reduce the maximum authentication attempts; and set up SSH timeouts. 

Another concept mentioned is key management. Security companies find thousands (if not millions) of floater keys on corporate networks all the time. That's because there often is no proper key management (SSH certificate authority, or other solution) and it results in SSH key sprawl - where the admins literally don't know where many of the keys are. 

Most of these concepts don't just apply to SSH servers, they apply to just about any "server" or mover of information. It could be a switch, router, PC, and of course, your actual traditional servers. 

And these security techniques only scratch the surface of what can be accomplished. Always be looking to improve on the security of your servers. 

---

That's it for the lab guide for Day 2! Hope you enjoyed it. 


