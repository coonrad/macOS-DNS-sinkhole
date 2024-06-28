# macOS-DNS-sinkhole
Use dnsmasq on macOS to sinkhole DNS traffic to specified domains.

This configuration has some limitations to a standard deployment of dnsmasq. The only DNS traffic directed to dnsmasq is the domains matched in `/etc/resolver`. There doesn't seem to be a simple or easy solution to direct all DNS traffic to dnsmasq in macOS. Despite this limitation it is very easy and useful to be able to block DNS to any configured domains.

## Setup

Install dnsmasq:

```bash
# Macports
port install dnsmasq
# Homebrew
brew install dnsmasq
```

This configuration uses paths for Macports. Adjust your config accordingly for Homebrew.

Create directories.

```bash
sudo mkdir -p /opt/local/etc/dnsmasq/
sudo mkdir /etc/resolver
```

Edit `/opt/local/etc/dnsmasq.conf`.

```bash
rebind-localhost-ok
stop-dns-rebind
strict-order
domain-needed
bogus-priv
no-hosts
dns-forward-max=5000
cache-size=10000
log-queries
log-facility=/var/log/resolver.log

listen-address=127.0.0.1

conf-file=/opt/local/etc/dnsmasq/domains
```

Edit `/opt/local/etc/dnsmasq/domains`

```bash
# custom domain block list
# each domain must have a matching record in /etc/resolver

# facebook
local=/facebook.com/
local=/facebook.net/
local=/fb.com/
```
