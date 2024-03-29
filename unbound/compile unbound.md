Read this first to see where to change folder names to have things in `/var/sbin` or `/var/bin`

https://www.systutorials.com/docs/linux/man/8-unbound-anchor/

## Compile, configure and install Unbound for Pi-hole on Raspbian 10 (buster)
with libevent for many outgoing ports (outgoing-range, num-queries-per-thread)
with libhiredis for Redis as backend (cachedb)

## Requirements
there is no Unbound installed via the package management of the operating system

### System changes (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo nano /etc/sysctl.conf`

```fs.file-max = 524288
net.core.rmem_default = 4194304
net.core.rmem_max = 4194304
net.core.wmem_default = 4194304
net.core.wmem_max = 4194304
net.core.somaxconn = 256
net.core.optmem_max = 32768
net.core.netdev_max_backlog = 1024
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 512
net.ipv4.tcp_slow_start_after_idle = 0
```


`sudo sysctl -p`

`sudo nano /etc/security/limits.conf`

```
root    soft    nofile  100000
root    hard    nofile  100000
*       soft    nofile  25000
*       hard    nofile  25000
```

## Create Unbound group & user (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo groupadd -g 991 unbound`

`sudo useradd -g unbound -u 991 -M -d /etc/unbound -s /usr/sbin/nologin unbound`

## Install needed packages (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo apt-get update && sudo apt-get upgrade`

`sudo apt-get install build-essential bison flex libssl-dev libexpat1-dev libevent-dev ca-certificates dnsutils libprotobuf-c-dev libprotobuf-c1 protobuf-c-compiler`

## Enable Unbound logrotate (ONLY ON FIRST INSTALLATION/COMPILATION) --

`sudo nano /etc/logrotate.d/unbound`

```
/var/log/unbound/unbound.log {
compress
delaycompress
missingok
daily
notifempty
minsize 500k
rotate 2
copytruncate
create 644 unbound unbound
}
```

`sudo shutdown -r now`

## Create Unbound data folder (ONLY ON FIRST INSTALLATION/COMPILATION) --

`sudo mkdir /etc/unbound/`

`sudo chown unbound:unbound /etc/unbound/`

## Update root.hints file

`sudo wget https://www.internic.net/domain/named.root -O /etc/unbound/root.hints`

`sudo chown unbound:unbound /etc/unbound/root.hints`

## Download Unbound sources

```
cd /tmp
unboundversion=unbound-1.12.0
wget https://nlnetlabs.nl/downloads/unbound/$unboundversion.tar.gz
tar xzf $unboundversion.tar.gz
cd $unboundversion
```

## Configure/Compile/Install Unbound

`sudo apt-get update && sudo apt-get upgrade`

`sudo update-ca-certificates`

`sudo ./configure --prefix=/usr --sysconfdir=/etc --with-conf-file=/etc/unbound/unbound.conf --with-username=unbound --with-pthreads --with-libevent --enable-cachedb --disable-static`

`sudo apt-get install libnghttp2-dev`

`sudo make`

If Unbound is already installed and running, stop it:

`sudo systemctl stop unbound.service`

`sudo make install`

## Unbound configuration file (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo rm /etc/unbound/unbound.conf`

`sudo nano /etc/unbound/unbound.conf`

```
server:
# mandatory settings - do not change
tls-cert-bundle: "/etc/ssl/certs/ca-certificates.crt"
chroot: "/etc/unbound"
username: "unbound"
directory: "/etc/unbound"
logfile: "unbound.log"
root-hints: "root.hints"
pidfile: "unbound.pid"
auto-trust-anchor-file: "root.key"
# end of mandatory settings

# recommended settings
outgoing-range: 8192
num-queries-per-thread: 4096
so-rcvbuf: 4M
so-sndbuf: 4M
access-control: 127.0.0.0/8 allow
# end of recommended settings

verbosity: 1
interface: 127.0.0.1@5335

remote-control:
control-enable: yes
control-interface: 127.0.0.1

auth-zone:
# local copy of the DNS root zone (hyperlocal)
# Use servers that provide zone transfer of the root zone (AXFR), see
# https://root-servers.org/faq.html and
# https://www.dns.icann.org/services/axfr/
name: "."
master: 192.5.5.241 # f.root-servers.net
master: 2001:500:2f::f # f.root-servers.net
master: 192.0.32.132 # lax.xfr.dns.icann.org
master: 192.0.47.132 # iad.xfr.dns.icann.org
master: 2620:0:2d0:202::132 # lax.xfr.dns.icann.org
master: 2620:0:2830:202::132 # iad.xfr.dns.icann.org
# Additional download via URL:
#url: "https://www.internic.net/domain/root.zone"
fallback-enabled: yes
for-downstream: no
for-upstream: yes
zonefile: "root.zone"
```

## Enable root & server keys

`sudo unbound-control-setup`

`sudo unbound-anchor`

## Check Unbound configuration file

`sudo /usr/sbin/unbound-checkconf /etc/unbound/unbound.conf`

## DNSSEC (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo /usr/sbin/unbound-anchor -r /etc/unbound/root.hints -a /etc/unbound/root.key -v`

## Set up SSL certificates for unbound-control (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo /usr/sbin/unbound-control-setup -d /etc/unbound/`

## Create Unbound service (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo nano /lib/systemd/system/unbound.service`

```
[Unit]
Description=Unbound DNS resolver
After=network.target
Before=nss-lookup.target pihole-FTL.service
Wants=nss-lookup.target

[Service]
Type=simple
ExecStartPre=-/usr/sbin/unbound-anchor -r /etc/unbound/root.hints -a /etc/unbound/root.key
ExecStart=/usr/sbin/unbound -c /etc/unbound/unbound.conf -d
ExecReload=/usr/sbin/unbound-control -c /etc/unbound/unbound.conf reload
ExecStop=/usr/sbin/unbound-control -c /etc/unbound/unbound.conf stop
PIDFile=/etc/unbound/unbound.pid
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```


## Install Unbound service (ONLY ON FIRST INSTALLATION/COMPILATION)

`sudo systemctl daemon-reload`

`sudo systemctl enable unbound.service`

If encountering an error where the service is masked:
If your service is masked it is symlinked to /dev/null
You can verify by:
`ls -l /etc/systemd/system/systemd-resolved.service`

To remove the mask simply run:
`sudo systemctl unmask unbound.service`

Then

`sudo systemctl daemon-reload`

`sudo systemctl enable unbound.service`

## Start Unbound

`sudo chown unbound:unbound /etc/unbound/*`

`sudo chown root:root /etc/unbound/unbound.conf`

`sudo systemctl start unbound.service`

## Check if Unbound is running

`sudo systemctl status unbound.service`

## create and check the log file
`sudo mkdir -p /var/log/unbound`

`sudo touch /var/log/unbound/unbound.log`

`sudo chown unbound /var/log/unbound/unbound.log`

`tail -n 20 /var/log/unbound/unbound.log`

## Test DNSSEC validation

`dig sigfail.verteiltesysteme.net @127.0.0.1 -p 5335`

`dig sigok.verteiltesysteme.net @127.0.0.1 -p 5335`

The first command should give a status report of SERVFAIL and no IP
address. The second should give NOERROR plus an IP address.

## Cleanup

`cd /tmp`

`sudo rm -rf $unboundversion`

`sudo rm $unboundversion.tar.gz`

`unboundversion=`

To compile and install a new Unbound version follow the how-to from top to bottom again, skip the "ONLY ON FIRST INSTALLATION/COMPILATION" steps
