# nftables lab

This lab demonstrates how to install and configure nftables on a Debian Server. (Other distros will work in a similar fashion.) Make sure that you can connect to the server from another computer. 

For this lab I will be working as root. If you decide to work as a standard Linux user, make sure the user is part of sudoers and issue the `sudo` command before all commands listed below.

> It is recommended that you work directly within the server's console (and not connect via SSH). 

## Install nftables

Run the following command:

`apt install nftables`

> Note: It may already be installed.

## Start and Enable nftables

Run the following command:

`systemctl --now enable nftables`

Verify that nftables is started and enabled with:

`systemctl status nftables`

You should see results similar to the following:

```console
root@debian12-test:~# systemctl status nftables
● nftables.service - nftables
     Loaded: loaded (/lib/systemd/system/nftables.service; enabled; preset: enabled)
     Active: active (exited) since Wed 2024-01-24 13:01:39 EST; 1min 46s ago
       Docs: man:nft(8)
             http://wiki.nftables.org
    Process: 236 ExecStart=/usr/sbin/nft -f /etc/nftables.conf (code=exited, status=0/SUCCESS)
   Main PID: 236 (code=exited, status=0/SUCCESS)
        CPU: 10ms

Jan 24 13:01:39 debian12-test systemd[1]: Finished nftables.service - nftables.
Notice: journal has been rotated since unit was started, output may be incomplete.
```

## Test the Server from a Remote System

From a separate system, try pinging the server and SSH'ing into it. Make sure that you can do so and that no other firewalls or protective software is running on the server.

## Understand nftables Terminology

nftables works off of the **Tables > Chains > Rules** concept which is quite common for firewalls. Remember this as you build your nftables!

Examine the default (insecure) ruleset in nftables:

`nft list ruleset`

You will note that everything inbound and outbound is open.

## Configure nftables

Here we will lock down the server.

All commands can be run starting with `nft`, but to make things easier, access the nft interactive shell:

`nft -i`

That should put you in the nft prompt:

`nft>_`

> Note: At this point, you can omit `nft` from your commands.

Now start building nftables for full security:

- Create a new table named "ports_table"

`add table inet ports_table`

- View the new table

`list ruleset`

You should see the new table listed below the original one, similar to the following:

```console
nft> list ruleset
table inet filter {
        chain input {
                type filter hook input priority filter; policy accept;
        }

        chain forward {
                type filter hook forward priority filter; policy accept;
        }

        chain output {
                type filter hook output priority filter; policy accept;
        }
}
table inet ports_table {
}
```

- Create a secure chain called "input"

`add chain inet ports_table input { type filter hook input priority 0 ; policy drop ; }`

This will lock down the server. Try pinging or ssh'ing to the server from a remote system. You should not be able to do so. 

> Note: If you were SSH'd into the server you will have lost your connection. That's why I recommend working in the console for this lab.

At this point, we have a system that cannot be accessed from the outside. In addition, the system cannot make any outbound connections.

Let's show how to open a single port (or service) by adding a rule to the ruleset.

- Create a rule that allows SSH

`add rule inet ports_table input tcp dport 22 accept`

List the ruleset to see your newly modified table/chain/rule.

Your nft ruleset should now look similar to the following: 

```console
table inet filter {
        chain input {
                type filter hook input priority filter; policy accept;
        }

        chain forward {
                type filter hook forward priority filter; policy accept;
        }

        chain output {
                type filter hook output priority filter; policy accept;
        }
}
table inet ports_table {
        chain input {
                type filter hook input priority filter; policy drop;
                tcp dport 22 accept
        }
}
```

> Note, you could also list the service name. For example, you could ave used "ssh" instead of "22".

Now, you should be able to ssh into the system. Attempt to do so from a remote computer. However, all other types of connections will be dropped.

## Save the Configuration

This nftables configuration is volatile. To make it persistent, we have to save it. 

First, backup the current nftables configuration:

- Exit out of the nft shell by typing `quit`.
- Examine the current nftables configuration:

  `cat /etc/nftables.conf`

  This should show the original configuration with no security.

- Backup the current configuration

  `cp /etc/nftables.conf nftables.conf.bak`

  > IMPORTANT! Don't skip this step! This will now be available to you so you can restore the original configuration later.

- Save the new configuration:

  `nft list ruleset > /etc/nftables.conf`

- Restart the nftables service

  `systemctl restart nftables`

- Reboot the server

- Check the ruleset and verify that it is persistent:

  `nft list ruleset`

- Attempt to ssh into the server. You should still be able to.
  
> Play with it! Learn nftables!

## Restore the Original Configuration

- Copy the backed up original config to /etc:

  `cp nftables.conf.bak /etc/nftables.conf`

- Restart nftables

  `systemctl restart nftables`

- Check that the original configuration is now running:

  `nft list ruleset`

You should see the original (insecure) configuration.

---

## Great work! 😄

For more information, consider the following sources:

https://wiki.nftables.org/wiki-nftables/index.php/Simple_rule_management

https://wiki.archlinux.org/title/nftables