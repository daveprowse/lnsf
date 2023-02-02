# Part V: User Security

In Linux you have the root user and everyone else, known as standard users. The root user has complete control over the system. Standard users can have varying levels of power depending on what you assign. 

When logged in and using a terminal a standard user will see the $ at the end of the prompt, as shown here:

`user@deb51:~$`

However, the root user login prompt will show the # sign:

`root@deb51:~#`

> Warning! Though we use the root account a lot in these labs, it is important to limit root usage in the field, and instead set up a standard user account as an administrator of the system. More on that later in this lab guide.

**Example "You try it!"**
	
  Go ahead and log in to your system as a regular account now. Then, login as the root account by typing `su -` and the password of the root user. Notice the difference between the two types of accounts.

### Lab 5-1
**Working with passwords**

In this lab we'll create a new account and attempt to assign a new password to the account. In so doing, we will see some of the basic password rules that are applied to user accounts by the operating system. This is shown on a Debian server.

- Create a new account

	As root, create a new account named *sysadmin*.

	`adduser sysadmin`

	Assign the password *test* to the account and confirm it. You can fill out the rest of the automated form if you wish or just press enter to bypass them. Here's an example of what it looks like on a Debian server:

	```
	root@deb51:~# adduser sysadmin
	Adding user `sysadmin' ...
	Adding new group `sysadmin' (1001) ...
	Adding new user `sysadmin' (1001) with group `sysadmin' ...
	Creating home directory `/home/sysadmin' ...
	Copying files from `/etc/skel' ...
	New password: 
	Retype new password: 
	passwd: password updated successfully
	Changing the user information for sysadmin
	Enter the new value, or press ENTER for the default
		Full Name []: Bob User
		Room Number []: 8
		Work Phone []: 5551212
		Home Phone []: 5551212
		Other []: test user
	Is the information correct? [Y/n] y
	root@deb51:~# 
	```

- Login as the new *sysadmin* account

	You can either logout of root and then login as sysadmin, or you can type:

	`su - sysadmin` which will start a new login shell for that account.

- Attempt to change the password:

	Type `passwd` to change the sysadmin account password. You will be prompted for the current password. 

	Now enter the following new password: *test1* and confirm it. 

	The system should respond by saying that you must choose a longer password. That's because Debian's default password policy for minimum length is 6 characters. 

- Attempt to change the password again:

	Type `passwd` again. This time, try the password *test12*. Although it meets the minimum requirements, it will fail again. The system will tell you that "Bad: new and old password are too similar". That's because the new password has the same characters as the old password. 

- Attempt to change the password a third time:

	Type `passwd` once again. This time, try the password *hello123*. This should work. 

	> Note:	Is *hello123* a secure password? Not really, by today's standards. At the bare minimum, passwords should be 8-10 characters, with a higher level of complexity. Length is very important. While the password in question is 8 characters, it just scrapes by in today's insecure world. It only uses lowercase letters and its only 8 characters. A more secure system would require passwords that are 16 characters minimum and use uppercase letters, lowercase letters, numbers, and special characters. 		For more information about password security, see the NIST SP 800-63B *Digital Identity Guidelines* document at [this link](https://pages.nist.gov/800-63-3/sp800-63b.html) 

	> Tip: Need a quick way to generate a password in Linux? If you have openSSL installed it's easy:

		`openssl rand -base64 24`
    
    This will generate a 24 character password for you. However, I recommend using a local password vault, such as [KeePass](https://keepass.info/download.html).

- Remove the sysadmin account

	We have an account with a weak password that we really don't need. As a sysadmin, always remember to clean up after your work. In this case, we'll remove the sysadmin account and any associated home directories.

	Logout of the sysadmin account and login as root. If you previously used the `su - sysadmin` command to login as sysadmin, you can simply type `logout`, or `exit` and you will be returned to the root shell. Otherwise, make sure you are logged in as root.

	Delete the sysadmin account:

	`deluser sysadmin`

	Then, delete the home directories for sysadmin:

	`rm -r /home/sysadmin`

	Check it with the following command:

	`ls /home`

	You should not see a sysadmin directory there any longer. An example of the entire removal process is shown below:

	```
	sysadmin@deb51:~$ exit
	logout
	root@deb51:~# deluser sysadmin
	Removing user `sysadmin' ...
	Warning: group `sysadmin' has no more members.
	Done.
	root@deb51:~# rm -r /home/sysadmin
	root@deb51:~# ls /home
	user
	root@deb51:~# 
	```

Great job, that concludes this lab!

### Lab 5-2
**su, sudo, and sudoers**

This lab demonstrates how to work with the `su`, `su -`, and `sudo` commands. It also shows how to make a user an administrator of the system. 

su is short for substitute user. If you ever need to login as a different user, the su command facilitates this. So for example, `su sysadmin` or `su - sysadmin` will login to the sysadmin account. One difference between the two is that `su` leaves you in the current directory that you were working in. `su -` puts you in the home directory of the user you are logging in as. 

sudo has multiple meanings. First, sudo is a program that runs on Linux systems which associates permissions with users and groups. sudo is also a group that a user can be added to in Debian/Ubuntu and similar systems. By default, once added to the sudo group, the user becomes an administrator. Finally, sudo is a command. When it precedes another command it allows the command to be run as an administrator. For example, `apt update` requires sudo rights. So for a typical user who acts as an admin to do an update, that user must type `sudo apt update`. Any command that requires administrative privileges needs to be preceeded by sudo. The only case where this is not true is if you are logged in as root. 

Users who have been given administrative privileges are known as a *sudoers*. That name is also the name of the sudo permissions file. 

Let's demonstrate all this now. Once again, this is demonstrated on a Debian server currently logged in as root.

- Use `su` to access another account

	Type `su sysadmin`. When you do so, it logs in as sysadmin, with no password required because you were already logged in as root. It leaves you in the same directory. Example:

	```
	root@deb51:~# su sysadmin
	sysadmin@deb51:/root$ ls
	ls: cannot open directory '.': Permission denied
	sysadmin@deb51:/root$ 
	```

	Note that we are logged in as sysadmin, but it placed us in the /root directory. That is the working directory for the root account. With `su sysadmin` we didn't actually open a new login shell. And we are restricted. If we type `ls` we will get a permission denied message because the sysadmin account cannot do anything in the root directory.

	Type `logout` or `exit` to logout of the sysadmin account and return to root.

- Use `su -` to access that same account

	Now type `su - sysadmin`. Typing `ls` works (though there is no content). Now type `pwd`. When you do so you will find that the default directory is /home/sysadmin as shown below. 

	```
	root@deb51:~# su - sysadmin
	sysadmin@deb51:~$ ls
	sysadmin@deb51:~$ pwd
	/home/sysadmin
	sysadmin@deb51:~$ 
	```

	This does actually create a new login shell. This is generally the preferred method, though not always. 

	Logout of sysadmin and back to root.

	> Note:	To login as root, use `su` or `su -`. The name "root" is not necessary.

- Install sudo

	Many Linux distributions already have sudo installed. You can easily find out by typing `sudo`. If not installed, it will tell you the command is not recognized. Debian (running as a server) does not have sudo installed by default. To install it, as root, type the following:

	`apt install sudo`

- Attempt to use sudo

	Login as sysadmin: `su - sysadmin`

	Type `apt update`. It won't work, because the sysadmin account would need to use the sudo command first.

	Type `sudo apt update`. It still won't work because the sysadmin account is not yet an actual administrator. It has not been added to the sudo group.

	Example below:

	```
	root@deb51:~# su - sysadmin
	sysadmin@deb51:~$ apt update
	Reading package lists... Done
	E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
	E: Unable to lock directory /var/lib/apt/lists/
	W: Problem unlinking the file /var/cache/apt/pkgcache.bin - RemoveCaches (13: Permission denied)
	W: Problem unlinking the file /var/cache/apt/srcpkgcache.bin - RemoveCaches (13: Permission denied)
	sysadmin@deb51:~$ sudo apt update
	sudo: unable to resolve host deb51: No address associated with hostname

	We trust you have received the usual lecture from the local System
	Administrator. It usually boils down to these three things:

	    #1) Respect the privacy of others.
	    #2) Think before you type.
	    #3) With great power comes great responsibility.

	[sudo] password for sysadmin: 
	sysadmin is not in the sudoers file.  This incident will be reported.
	```

	Logout of sysadmin and back into root.

- Add the sysadmin account to sudo and test

	So, as it stands, the sysadmin account is an admin in name only - not in actual privilege. We have to add that privilege. We do that by making sysadmin a sudoer. You could also phrase that as: adding sysadmin to the sudo group. This group was created automatically when we installed the sudo program.

	Issue the following command:

	`usermod -aG sudo sysadmin`

	That modifies the sysadmin account, and adds it to the sudo group. Now, it should have administrative privileges.

	Login as sysadmin again: 

	`su - sysadmin`

	And attempt to update the system:

	`sudo apt update`

	The first time you use the sudo command, the system should request the user's password. Type it. The command should now work and the program will tell us how many packages can be updated. That's it, sysadmin now has administrative privileges. But only as long as the command sudo precedes the administrative command. 

- View the sudoers file

	The sudoers file is where we can specify privileges for groups and individual users. It is located at /etc/sudoers. However, the recommended way to modify the file is to use the `visudo` command. But this command will require administrative access, so as sysadmin, we would have to add sudo before the command. To do this, simply type:

	`sudo visudo`

	**Example of visudo on a Debian Server**

		```
		#
		# This file MUST be edited with the 'visudo' command as root.
		#
		# Please consider adding local content in /etc/sudoers.d/ instead of
		# directly modifying this file.
		#
		# See the man page for details on how to write a sudoers file.
		#
		Defaults	env_reset
		Defaults	mail_badpass
		Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

		# Host alias specification

		# User alias specification

		# Cmnd alias specification

		# User privilege specification
		root	ALL=(ALL:ALL) ALL

		# Allow members of group sudo to execute any command
		%sudo	ALL=(ALL:ALL) ALL

		# See sudoers(5) for more information on "#include" directives:

		#includedir /etc/sudoers.d
		```

	Looking toward the bottom of the file, you will note that the root account has ALL permissions. So does the sudo group (referenced as %sudo). But this can be changed. The sudo group can be limited, or new groups can be created with their own permissions. Individual accounts can also be assigned particular permissions to the system.

	> Note:	In Fedora/RHEL/CentOS systems, the %wheel group takes the place of %sudo. This applies to the sudoers file and if you want to apply administrative permissions to a user account. So for example: 
		
		`usermod -aG wheel sysadmin`

	> Tip:
		By default sudo asks for the password of the user. It won't ask again for 5 minutes. To increase this timeout, add the following line to the sudoers file:
	
		`Defaults:sysadmin timestamp_timeout=30`
    
    That changes the timeout to 30 minutes for sysadmin. Modify it as you wish, but it is not suggested to modify it to 0 (which is infinite). 

	This is just a primer. There is plenty more to users, groups, permissions, sudo, and so on. But this is the core of what is happening on a Debian or Ubuntu system. It's one of the more complicated concepts in the Linux operating system. If you can understand this, then you can understand just about anything in Linux.

## ***Users and Passwords Part II***

The default password settings are weak in most distributions of Linux. To secure the system, the settings should be modified. But first, let's do some analysis of a Debian server. Log in as root. First we'll look at the list of user accounts. This can be found in the /etc/passwd file. Even though the name is "passwd", it doesn't store passwords. View it now:

`cat /etc/passwd`

You should see a list of user accounts and system accounts. The root account is in the beginning of the list. Newly created accounts are at the end. For example, the line item that has the sysadmin account will look similar to this:

```
sysadmin:x:1001:1001:,,,:/home/sysadmin:/bin/bash
```

This shows us that the username is sysadmin. "x" is an older setting, it used to show the password location but is not used anymore. 1001 shows us the User ID and the Group ID. Then we see the home directory and the shell being used (which is bash). So keep in mind that the passwd file does not actually show password information. It only shows user account information. 

To see password-based information, we use the /etc/shadow file. View it now:

`cat /etc/shadow`

This also shows a list of accounts, but now we get password-based information. For example:

```
sysadmin:$6$uIxm1E1ZzM1i6wR9$nafysqSgqg6rJNE2IlPGK4tPgMWBSIcpFuqhKtPWRZKYyrBs1m2bUSt/PGDfZWDPCkisK7rhk1Qg2Lpjcpxdp/:18729:0:99999:7:::
```

This shows us sysadmin, then $6 which means that the password is being cryptographically hashed with the SHA-512 protocol. Then the actual hashed password is displayed. Identifying the type of cryptographic hash is important. For example, if it said $1, then that would mean the server is using MD5 which is deprecated and not recommended. 
!!! note
	You can also view the type of hash algorithm being used by accessing the /etc/pam.d/common-password file. Look for the line similar to this:

	```
	password	[success=1 default=ignore]	pam_unix.so obscure sha512
	```
	
	This shows once again that we are using SHA512. 

### Lab 5-3
**Working with Password Settings**

In this lab we'll show how to change the default password settings and criteria on a Debian server as root.

- Use `chage` to change password settings

	Type the following command:	

	`chage sysadmin`

	This command allows us to change the age requirements for a password as shown below:

	```
	root@deb51:~# chage sysadmin
	Changing the aging information for sysadmin
	Enter the new value, or press ENTER for the default

		Minimum Password Age [0]: 0
		Maximum Password Age [99999]: 365
		Last Password Change (YYYY-MM-DD) [2021-04-12]: 
		Password Expiration Warning [7]: 
		Password Inactive [-1]: 
		Account Expiration Date (YYYY-MM-DD) [-1]: 2021-12-31
	root@deb51:~# 
	```

	Here the maximum password age was changed from 99999 days to 365 days. Also, there is an account expiration date at the end of the year 2021. After December 31, 2021, the user account sysadmin will not be able to login. The other settings were not modified. 

	!!! tip
		While this is a good command to know, it is not used quite as often today. It has a lot of additional options which can be useful though. For example, to set a user's password to expire (make it a zero-day password) you can issue the following command:

		`chage -d 0 sysadmin`

		This will require the sysadmin account to change its password on next login. 

- Modify password minimum length and criteria

	In Debian and Ubuntu you can change the minimum password length and other criteria in /etc/pam.d/common-password. Open the file now:

	`vim /etc/pam.d/common-password`

	Find the line that we located earlier:

	```
	password	[success=1 default=ignore]	pam_unix.so obscure sha512
	```

	To change the minimum password length, add `minlen=12` to the end of that line so that it looks like this:

	```
	password	[success=1 default=ignore]	pam_unix.so obscure sha512 minlen=12
	```

	Now, instead of a 6 character password minimum, Debian will require 12 character passwords from users. Increase the number as you see fit, or based on organizational policies!

	You can also add criteria such as requiring uppercase letters and special characters. First, on Debian and Ubuntu, password quality has to be installed. 

	`apt install libpam-pwquality`

	Once done, the common-password file should now show a new line above the previous line that we modified that says "pam_pwquality.so".

	Now you can have additional options to the new line ending in "pam_pwquality.so":

	- To require uppercase letters: `ucredit=1`

	- To require lowercase letters: `dcredit=1`

	- To require special (or other) characters: `ocredit=1`

	- To require that a certain amount of password criteria be included: `minclass=2`

	- To specifically require a particular amount of characters, use the minus sign. For example, `ocredit=-1`. Note the *-1* instead of a 1. 

	Here's an example of a configuration that requires 12 characters, a minimum of 1 uppercase, 1 lowercase, and *2* special characters. 

	```
	# here are the per-package modules (the "Primary" block)
	password	requisite			pam_pwquality.so retry=3 ucredit=1 dcredit=1 ocredit=-2 minclass=3
	password	[success=1 default=ignore]	pam_unix.so obscure use_authtok try_first_pass sha512 minlen=12
	```

	> Warning: Be sure to test this with a standard user account, and not the root account! It's easy to forget a password!

	> Note:	This portion of the lab will work differently on Fedora/RHEL/CentOS systems. There you use the Authselect tool. 

- Change the password expiration globally

	Previously, we changed the password expiration for a single user account with the `chage` file. Now, let's change it for all users.

	As root, open the /etc/login.defs file:

	`vim /etc/login.defs`

	Search for the line that says `PASS_MAX_DAYS`. Modify it as you see fit. Example below:

	```
	# Password aging controls:
	#
	#	PASS_MAX_DAYS	Maximum number of days a password may be used.
	#	PASS_MIN_DAYS	Minimum number of days allowed between password changes.
	#	PASS_WARN_AGE	Number of days warning given before a password expires.
	#
	PASS_MAX_DAYS	365
	PASS_MIN_DAYS	0
	PASS_WARN_AGE	7
	```

	I thought that 274 years was a little too long. So here the PASS_MAX_DAYS setting was changed from 99999 days to 365 days. But this only applies to new accounts.

That wraps up this lab, and Part V of this lab guide. 