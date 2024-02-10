# ipblockd
## ip and network blocking based on syslog rules (or anything else you might imagine)

### INSTALLATION

> At the end of your iptables set (after all allow rules!), you will want
```
    -A INPUT -m ... -j ALLOW
    ...
    -A INPUT -m set --match-set ipblockd src -j DROP
    -A INPUT -m set --match-set ipblockd-nets src -j DROP
```
> same applies for IPv6:
```
    -A INPUT -m ... -j ALLOW
    ...
    -A INPUT -m set --match-set ipblockd-ipv6 src -j DROP
    -A INPUT -m set --match-set ipblockd-nets-ipv6 src -j DROP
```
**ipblockd.rsyslog.conf** is a rsyslog conf file to be placed in ``/etc/rsyslog.d/`` - if you use syslog-ng, it can do the same but you will have to write a similar file. Note that all filter conditions are in this file. Restart rsyslog after placing the file and ```# mkfifo /var/spool/rsyslog/ipblockd``` before you do so.

**ipblockd.service** is the systemd unit file to be placed in ``/etc/systemd/system/``. You will want to
```
    # systemctl daemon-reload
    # systemctl enable ipblockd.service
    # systemctl start ipblockd.service
```

You will need ```logger``` from [util-linux](https://github.com/util-linux/util-linux), obviously ```ipset``` from [netfilter.org](https://ipset.netfilter.org/) and some version of perl with installed ```Regexp::IPv4``` and ```Regexp::IPv6``` - most distributions come with all of these pre-installed anyway.

Be very careful! Anything which looks like an IP, either v4 or v6, which is piped into ```/var/spool/rsyslog/iblockd``` will be blocked by the firewall within a second or two. Be sure to adjust ```WHITELIST``` to contain at least your home country. Adjust ```BLACKLIST``` to your liking. An IP originating in one of the blacklisted countries will result in its entire ASN being blocked instead of just the IP itself. This cuts down on attacs dramatically.

The ipsets are created automatically. They have a default timeout of 24 days, so no blocking will ocurr for ever.

If you don't want to use systemd or have a different init system (System V init or upstart or ...) you may add an ampersand (```&```) to the ```main``` in the very last line of the script. This will detach it from the calling terminal having the same effect as running it with '&' on the command line.

### HINTS


> normal operation without using systemd:
```
    $ sudo ipblockd &
```

> run a verbose session without detaching from your console like this:
```
    $ sudo DEBUG=2 TIMEOUT='600' ipblockd 
```

> from a second terminal, dump some statistics to the log or STDOUT:
```
    $ sudo killall -USR1 ipblockd
```

> see what it's doing at runtime:
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

