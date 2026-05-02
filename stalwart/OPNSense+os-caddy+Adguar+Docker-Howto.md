# HowTo run

- Stalwart (Mail, WebDAV, CalDAV, CardDAV) suite in Rust
- behind OPNSense firewall using
- os-caddy Caddy as reverse proxy
- AdGuard as DNS filter for LAN
- External (Public DNS) server for
- domain mydomain.de

# Assumptions

 - OPNSense has WAN-Port with dhcp IPv4 + IPv6
 - OPNSense has LAN-Port 192.168.1.0/24
 - OPNSense has SERVER-Port 192.168.5.0/24
 - AdGuard for LAN-network only
 - Server for Stalwart running on 192.168.5.13/24 standard Linux

# Preparations

## Public DNS
 - Configure "mail.mydomain.de" to point to OPNSense WAN-address
 - Configure "mx" for "@.mydomain.de" weight 10 to point to mail.mydomain.de
 - Configure "txt" for "@.mydomain.de" with content "v=spf1 mx -all"
 - Configure "caa" for "@.mydomain.de" with content "0 issue letsencrypt.org"
 - Configure "srv" for "_imaps._tcp.mydomain.de" with content "0 1 993 mail.mydomain.de"
 - Configure "srv" for "_submission._tcp.mydomain.de" with content "0 1 587 mail.mydomain.de"
 - Configure "srv" for "_submission**s**._tcp.mydomain.de" with content "0 1 465 mail.mydomain.de"
 - Configure "cname" for "autoconfig.mydomain.de" to "mail.mydomain.de"
 - Configure "cname" for "autodiscover.mydomain.de" to "mail.mydomain.de"
 - Configure "cname" for "mta-sts.mydomain.de" to "mail.mydomain.de"

Wait for DNS updates to propagate (up multiple hours)

## OPNSense

### Prepare Firewall aliasses
 - Define Alias "PG_Mail" (Port Group) for ports (content) 25 (SMTP), 110 (POP3), 143 (IMAP), 465 (SMTPS), 587 (SMTPS), 993 (IMAPS), 995 (POP3S)
 - Define Alias "H_Mailserver" (Host) for IP 192.168.5.13

### Define Firewall rules
 - Define new rule on WAN-Port: Pass IPv4+IPv6/TCP from any/any to WAN_address/PG_Mail (incoming mail ports)
 - Define new rule on LAN-Port: Pass IPv4/TCP from LAN-network/any to H_Mailserver/PG_Mail (local mail access)
 - Define new rule on SERVER-port: Pass IPv4/TCP from SERVER-network/any to any/PG_Mail (outgoing mail ports)

### AdGuard
 - Add Filter >> DNS-Rewrites for "mail.mydomain.de" to 192.168.5.13
   
### Caddy (Reverse Proxy)
 - Login to OPNSense >> Services >> Caddy >> Reverse Proxy
 - Create new domain "mail.mydomain.de" - Access List "None"
 - Create new handler for "mail.mydomain.de", enable Advanced Mode, set
   - Access >> Access List >> LAN-network only
   - Handler >> Path to "/admin/*"
   - Protocol to https
   - Upstream domain to 192.168.5.13
   - Upstream port to 443
 - Create new handler for "mail.mydomain.de", set
   - Access >> Access List to None
   - Protocol to https
   - Upstream domain to 192.168.5.13
   - Upstream port to 443

This will limit access to /admin/ only from LAN-AccessList and allow all others.

### Caddy (Layer4 proxy)
 - Login to OPNSense >> Services >> Caddy >> General Settings and enable Layer4 proxy
 - Navigate to OPNSense >> Services >> Caddy >> Layer4 Proxy and add Layer4 Routes with increasing sequence numbers
   - Routing Type global/TCP, Local Port 25, Matchers ANY, Proxy Procotol v2, Upstream Domain 192.168.5.13, Upstream Port 25
   - Routing Type global/TCP, Local Port 110, Matchers ANY, Proxy Procotol v2, Upstream Domain 192.168.5.13, Upstream Port 110
   - Routing Type global/TCP, Local Port 143, Matchers ANY, Proxy Procotol v2, Upstream Domain 192.168.5.13, Upstream Port 143
   - Routing Type global/TCP, Local Port 465, Matchers ANY, Proxy Procotol v2, Upstream Domain 192.168.5.13, Upstream Port 465
   - Routing Type global/TCP, Local Port 587, Matchers ANY, Proxy Procotol v2, Upstream Domain 192.168.5.13, Upstream Port 587
   - Routing Type global/TCP, Local Port 993, Matchers ANY, Proxy Procotol v2, Upstream Domain 192.168.5.13, Upstream Port 993
   - Routing Type global/TCP, Local Port 995, Matchers ANY, Proxy Procotol v2, Upstream Domain 192.168.5.13, Upstream Port 995

## Mail server host
 - Install docker-compose, docker.io
 - Prepare `/opt/stacks/stalwart/docker-compose.yml` with contents

````
services:
  stalwart-mail:
    image: stalwartlabs/stalwart:latest
    volumes:
      - ./stalwart-data:/opt/stalwart
      - ./stalwart-etc/:/etc/stalwart
      - ./stalwart-lib:/var/lib/stalwart
    restart: unless-stopped
    ports:
      - 443:443   # Webfrontend
      - 8080:8080 # Webfrontend HTTP
      - 25:25     # SMTP
      - 465:465   # SMTPS
      - 587:587   # SMTPS
      - 993:993   # IMAPS
      - 4190:4190 # Spamfilter
    environment:
      - TZ=Europe/Berlin
````
  - Create group stalwart with GID = 2000
  - Create user stalwart with UID = 2000
  - Create directories `/opt/stacks/stalwart/stalwart-{data|lib|etc}`
  - Create directory `/opt/stacks/stalwart/stalwart-data/certs`
  - chown -R the three directories to stalwart:stalwart
  - chmod -R u=rwx,g=rws,o= the three directories

### OPNSense & Caddy (passwordless ssh to stalwart server)
 - ssh Login to OPNSense, use 8 for shell
 - Setup passwordless scp to Mailserver for user stlwart by creating ssh-key and copying public key to authorized host on Server
 - Validate that `ssh -i stalwart stalwart@192.168.5.13` works without password ("stalwart" is the ssh-key to be included and the user)
 - Setup a cronjob at 03:00 daily to copy `/var/db/caddy/data/caddy/certificates/acme-v02.api.letsencrypt.org-directory/mail.mydomain.de/mail.mydomain.de.{crt|key}` to `/opt/stacks/stalwart/stalwart-data/certs`
