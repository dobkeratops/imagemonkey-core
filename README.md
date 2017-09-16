# README #

## INFRASTRUCTURE ##

The following section contains some notes on how to set up your own instance to host `imagemonkey` yourself.
This should only give you an idea how you *could* configure your system. Of course you are totally free in choosing 
a different linux distribution, tools and scripts. 

Info: Some commands are distribution (Debian 9.1) specific and may not work on your system. 

* create a new user `imagemonkey`  with `adduser imagemonkey` 
* disable root login via ssh by changing the `PermitRootLogin` line in `/etc/ssh/sshd_config` to `PermitRootLogin no`)
* block all ports except port 22, 443 and 80 (on eth0) with: 
```
#!bash

iptables -P INPUT DROP && iptables -A INPUT -i eth0 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT
```

* allow all established connections with:

```
#!bash

iptables -A INPUT  -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

* allow all loopback access with:
```
#!bash
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
```

* install `iptables-persistent` to load firewall rules at startup
* save firewall rules with: `iptables-save > /etc/iptables/rules.v4`
* verify that rules are loaded with `iptables -L`
* install PostgreSQL
* edit `/etc/postgresql/9.6/main/postgresql.conf` and set `listen_addresses = 'localhost'`
* restart PostgreSQL service with `service postgresql restart` to apply changes
* create database by applying schema `/env/postgres/schema.sql` with `psql -f schema.sql`
* create new postgres user `monkey` by executing the following in psql: 
```
CREATE USER monkey WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE imagemonkey to monkey;

```
* test if newly created user works with: `psql -d imagemonkey -U monkey -h 127.0.0.1`

* install nginx with `apt-get install nginx`
* install nginx-extras with `apt-get install nginx-extras`
* install letsencrypt certbot with `apt-get install certbot`
* add a A-Record DNS entry which points to the IP address of your instance
* run `certbot certonly` to obtain a certificate for your registered domain
* modify `conf/nginx/nginx.conf` and replace `imagemonkey.io` and `api.imagemonkey.io` with your own domain names, copy it to `/etc/nginx/nginx.conf` and reload nginx with `service nginx reload`
* install supervisor with `apt-get install supervisor`
* add `imagemonkey` user to supervisor group with `adduser imagemonkey supervisor`
* create logging directories with `mkdir -p /var/log/imagemonkey-api` and `mkdir -p /var/log/imagemonkey-web`

### Building Application ###
* install git with `apt-get install git`
* install golang with `apt-get install golang`
* clone repository
* set GOPATH with `export GOPATH=$HOME/go`
* set GOBIN with `export GOBIN=$HOME/bin`
* install all dependencies with `go get -d ./... `
* install API application with `go install api.go api_secrets.go common.go imagedb.go`
* install API application with `go install web.go web_secrets.go common.go imagedb.go` 
* copy `wordlists/en/misc.txt` to `/home/imagemonkey/wordlists/en/misc.txt`
* create donations directory with: `mkdir -p /home/imagemonkey/donations`
* copy `conf/supervisor/imagemonkey-api.conf` to `/etc/supervisor/conf.d/imagemonkey-api.conf`
* copy `conf/supervisor/imagemonkey-web.conf` to `/etc/supervisor/conf.d/imagemonkey-web.conf`
* run `supervisorctl reread && supervisorctl update && supervisorctl restart all`


## What's next? ##

### General ###
* API abuse prevention: I strongly believe that such a project only works out, if we keep the API as open as possible (i.e ideally accessible without any registration). However, without any kind of registration or API tokens, it's easily possible that malicious bots/people will destroy valid datasets by wrongly classifying a picture. 

Possible attempts to solve that: 
- implement an alerting mechanism that fires when a image rapidly changes it's validity
- add a "training phase" where users need to verify a few easy pictures first and only after they completed the "training phase" and have proven that their intentions are good then their votes are counted
- add an registration mechanism to make abusive voting less attractive (not my prefered option, but yeah...)

* remember already seen images to avoid that users verify a specific image twice

### Infrastructure ###
* currently there are a lot of manual steps involved to host your own instance of `imagemonkey`. There should be a script which automates that. What about a dockerizable image? 
* add a deployment script which makes deploying changes easier and less error prone.

### REST API ###
* make it possible to export images only when `probability > treshold`

### Sourcecode ###
* there is some duplicated code in `api.go` and `web.go` -> we should get rid of it
