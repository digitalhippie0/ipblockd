# ipblockd
## ip and network blocking based on syslog rules (or anything else you might imagine)

### PREREQUESITES

You will need ```logger``` from [util-linux](https://github.com/util-linux/util-linux), obviously ```ipset``` from [netfilter.org](https://ipset.netfilter.org/) and some version of perl with installed ```Regexp::IPv4``` and ```Regexp::IPv6``` - most distributions come with all of these pre-installed anyway.

If you want to see some statistics in the log, you will need bash v4 or above. I use associative arrays here which are new to bash since v4. They come in very handy. If your shell has a version <4 *ipblockd* will still run but all you get on SIGHUP is a short message in the log, sating that you need to update your shell. 

### INSTALLATION

#### PATH

*ipblockd* is meant to live in either /usr/sbin or /usr/local/sbin; it is written to be run by the root user. You may try to change IPSET_BIN to something like ```IPSET_BIN='/usr/bin/sudo /usr/sbin/ipset'``` and put an allow rule for your user in the sudoers file, but if you do so, you're on your own.

#### NAMED PIPE

Create a named pipe for *ipblockd* to listen on. 

```
    $ sudo mkfifo /var/spool/rsyslog/ipblockd
```

#### FIREWALL RULES

You will have to create the ipsets first, in order to be able to create firewall rules based on them. *ipblockd* will create the default set for you, just run it once prior to configuring the firewall.

At the end of your (IPv4) iptables set (after all allow rules!), you will want to add two rules:

```
    -A INPUT -m ... -j ALLOW
    ...
    -A INPUT -m set --match-set ipblockd src -j DROP
    -A INPUT -m set --match-set ipblockd-nets src -j DROP
```
same applies for IPv6:
```
    -A INPUT -m ... -j ALLOW
    ...
    -A INPUT -m set --match-set ipblockd-ipv6 src -j DROP
    -A INPUT -m set --match-set ipblockd-nets-ipv6 src -j DROP
```

#### FILTERING RULES

**ipblockd.rsyslog.conf** is a rsyslog conf file to be placed in ``/etc/rsyslog.d/`` - if you use syslog-ng, it can do the same but you will have to write a similar file. Note that all filter conditions are in this file. Restart rsyslog after placing the file and ```# mkfifo /var/spool/rsyslog/ipblockd``` before you do so.

#### SYSTEMD

**ipblockd.service** is the systemd unit file to be placed in ``/etc/systemd/system/``. You will want to
```
    # systemctl daemon-reload
    # systemctl enable ipblockd.service
    # systemctl start ipblockd.service
```

If you don't want to use systemd or have a different init system (System V init or upstart or ...) you may add an ampersand (```&```) to the ```main``` in the very last line of the script. This will detach it from the calling terminal having the same effect as running it with '&' on the command line.

#### DEFAULTS FILE

If you choose to use the provided systemd file as is, you may use the **ipblockd.defaults** file. On a debian system, this will live in ```/etc/defaults/```. On centOS and alike - I believe - this should be in ```/etc/sysconfig/```.

Any other method of setting these environmient variables is fine, too.

### CAVEATS

Be very careful! Anything which looks like an IP, either v4 or v6, which is piped into ```/var/spool/rsyslog/iblockd``` will be blocked by the firewall within a second or two. Be sure to adjust ```WHITELIST``` to contain at least your home country. Adjust ```BLACKLIST``` to your liking. An IP originating in one of the blacklisted countries will result in its entire ASN being blocked instead of just the IP itself. This cuts down on attacs dramatically.

The ipsets are created automatically. They have a default timeout of 24 days, so no blocking will ocurr for ever. Set a timeout of 0 to have essentially no timeout.

### HINTS


normal operation without using systemd:
```
    $ sudo ipblockd &
```

run a verbose session without detaching from your console like this:
```
    $ sudo DEBUG=2 TIMEOUT='600' ipblockd 
```

from a second terminal, dump some statistics to the log or STDOUT:
```
    $ sudo killall -USR1 ipblockd
```

see what it's doing at runtime:
```
    $ sudo journalctl -f -u ipblockd
```

### COPYRIGHT

&copy; 2024 Henrik Harms

ipblockd is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free
Software Foundation, either version 3 of the License, or (at your option)
any later version.

ipblockd is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General
Public License for more details.

You should have received a copy of the GNU General Public License along
with ipblockd. If not, see <https://www.gnu.org/licenses/>.

### AUTHOR

Henrik Harms, Vienna, Austria

