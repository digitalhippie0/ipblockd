# ipblockd
## ip and network blocking based on syslog rules (or anything else you might imagine)

### SYNOPSIS

ENVAR1=foo ENVVAR2=moo ipblockd
*there are no available options, everything is dictated by the environment, see ENVIRONMENT VARIABLES below*

### DESCRIPTION

Are you running a service on the public internet which has some kind of login exposed? Maybe, like me, a mail server with smtp, pop3(s) and imap(s) service? Do you observe lots and lots of *failed authentication attempts* originating in countries where your users never reside? Well. I got really annoyed by literally thousands of failures per day. Clearly, some botnet was desperately trying to brute force one of my users logins, most likely to use the break in order to donate even more **SPAM** to the world.

So I wrote a quick hack to stop this and kept refinig it. Unlike ```fail2ban``` from [https://github.com/fail2ban], my script actually does lookups for the originating networks (ASN) and creates rules for entire netblocks from certain countries. It will also block individual hosts from other countries. Also different is the way the blocking occurs. We use ipsets here, this has quite some advantages. An ipset containing millions of hosts can be matched with a single firewall rule and we do not clutter the firewall with individual rules. This also takes a lot of load off the netfilter stack.

**ipblockd** takes advantage of the builtin timeouts and counters in **ipset**, it handles IPv4 and IPv6 equally well. All logic to parse for relevant log lines is delegated to the logging daemon of your system (usually rsyslog or syslog-ng) - what we do here is listen to a named pipe and look out for IP addresses. It is written to be a modern systemd loaded service and comes with a proper service file. As of 2024, we can safely delegate much of the work required to keep a service running to systemd. Take a look at the HINTS section in the *INSTALL.md* file to see how to use ipblockd.

### ENVIRONMENT VARIABLES

All variables are optional. ipblockd has useable defaults for all of them. You *will* want to review the _BIN variables though and adapt them to match your system.
For a more complete documentation, see *ipblockd.defaults* or the *INSTALL.md* files.

IPSET_BASE
BLACKLIST
WHITELIST
PIPE
IPSET_BIN
LOGGER_BIN
BC_BIN
DEBUG
TIMEOUT
RESETSTATS

### FILES

*/etc/systemd/system/ipblockd.service*
*/etc/defaults/ipblockd*
*/etc/rsyslog.d/ipblockd.rsyslog.conf*
*/usr/local/sbin/ipblockd*
*/var/spool/rsyslog/ipblockd* (named pipe)

### SIGNALS

Sending a SIGUSR1 to the service will trigger a statistics dump which will appear in the log. Depending on the variable RESETSTATS statistics will be reset or not. Who may have thought ...
Sending a SIGHUP will terminate the main read loop and thus trigger a pipe reopening - just in case
Sending any of SIGINT, QUIT, TERM and so on will hopefully terminate the program ;)

### KNOWN BUGS

### BUG REPORTS

Report bugs to [https://github.com/digitalhippie0/ipblockd/issues]

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

Henrik Harms, Vienna (AT)
