# macOS-DNS-sinkhole
Use dnsmasq on macOS to sinkhole DNS traffic to specified domains.

This configuration has some limitations to a standard deployment of dnsmasq. The only DNS traffic directed to dnsmasq is the domains matched in `/etc/resolver`. There doesn't seem to be a simple or easy solution to direct all DNS traffic to dnsmasq in macOS. Despite this limitation it is very easy and useful to be able to block DNS to any configured domains.

