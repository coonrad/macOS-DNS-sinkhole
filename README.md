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

Create `/etc/resolver/` files for each domain.

```bash
printf "nameserver 127.0.0.1" | sudo tee /etc/resolver/facebook.com
printf "nameserver 127.0.0.1" | sudo tee /etc/resolver/facebook.net
printf "nameserver 127.0.0.1" | sudo tee /etc/resolver/fb.com
```

Load dnsmasq.

```bash
sudo port load dnsmasq
```

At this point dnsmasq is configured to listen on `127.0.0.1`. Any traffic for the domains in `/etc/resolver` will be directed to dnsmasq. This will result in `NXDOMAIN` and traffic to the specified domain will not resolve. You can verify this by doing `host` or `dig` commands against `127.0.0.1` or viewing the resolver log. (If you don't want to log queries to dnsmasq. Comment out `log-facilty` in the configuration file.)

```bash
Jun 27 23:59:33 dnsmasq[89]: query[A] facebook.com from 127.0.0.1
Jun 27 23:59:33 dnsmasq[89]: config facebook.com is NXDOMAIN
```

With some automation you should be able to block hundreds or thousands of domains. For instance this bash function will grep the domains from the domains file and populate an entry for each one to `/etc/resolver`.

```bash
function dnsmasq_setup() {

    # grep list of domains for /etc/resolver
    domains=$(grep -o -P '(?<=/).*(?=/)' /opt/local/etc/dnsmasq/domains)

    # create /etc/resolver file for each domain
    for i in $domains; do
        echo "nameserver 127.0.0.1" | sudo tee /etc/resolver/"$i" >/dev/null
    done

    # restart dnsmasq
    sudo port reload dnsmasq
}
```

