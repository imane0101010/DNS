# DNS
## Configuration DHCP server
### Prerequisites
make sure the DNS server has a static ip
update the repository index
### Installation DNS Server
```sh
sudo apt-get install -y bind9 bind9utils bind9-doc dnsutils
```
### Configuration DNS Server
#### Create zones
Let´s begin by creating a forward zone for the domain.
```sh
sudo nano /etc/bind/named.conf.local
```
#### Forward and Reverse Zones
The following is the forward zone entry and the reverse zone for the me.local domain in the named.conf.local file.
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-19%2014-53-38.png"></p> 

#### Create Zone Lookup file 

##### Forward Zone

Once zones are created, we will create zone data files for the forward zone and reverse zone.
```sh
sudo nano /etc/bind/forward.me.local.db
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-19%2014-53-08.png"></p> 

###### Reverse Zone

```sh
sudo nano /etc/bind/reverse.me.local.db
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-19%2014-52-47.png"></p> 

#### Check BIND configuration syntax

```sh
sudo named-checkconf
```
##### Forward Zones

```sh
sudo named-checkzone me.local /etc/bind/forward.me.local.db
```
##### Reverse Zones

```sh
sudo named-checkzone 0.168.192.in-addr.arpa /etc/bind/reverse.me.local.db
```
#### Restart DNS

```sh
sudo systemctl restart bind9
```
#### Status DNS

```sh
sudo systemctl status bind9
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-17%2009-01-37.png"></p> 

#### Verification

Let us add the new DNS Server ip adress in /etc/resolv.conf file using:
```sh
sudo nano /etc/resolv.conf
```
nameserver 192.168.1.10 (int this case)
To verify our DNS Server,we can use dig command to verify the reverse and forward lookup.
```sh
dig ns1.me.local
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/dns_ans2.png"></p> 

```sh
dig -x 192.168.1.10
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/dns_ans1.png"></p> 

## Configuration Master and Slave

### Master

#### Editing zones



```sh
sudo nano /etc/bind/named.conf.local
```
#### Forward and Reverse Zone

<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/e3e86937d7119bd2b9c1f176a847a153aa1a406c/DNS/MASTER_&_SLAVE/MASTER/DNS_MASTER3.png"></p>


#### Editing Lookup files

#### Forward Zone
```sh
sudo nano /etc/bind/forward.me.local.db
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/MASTER_&_SLAVE/MASTER/DNS_MASTER.png"></p>

#### Reverse Zone

```sh
sudo nano /etc/bind/reverse.me.local.db
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/MASTER_&_SLAVE/MASTER/DNS_MASTER1.png"></p>

### Slave
#### Prerequisites

Static IP address
Bind9 installed

#### Editing Zones

##### Forward and Reverse Zones

<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE1.png"></p>

##### Restart Bind9

```sh
sudo systemctl restart bind9
```
##### Verification

Let us see if the forward and reverse zone file is already in /var/cache/bind

```sh
sudo ls -l /var/cache/bind
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE2.png"></p>

Then let´s add the DNS slave and master ip address in /etc/resolv.conf
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE.png"></p>
Now,let´s check the configuration using dig command

```sh
dig ns2.me.local @192.168.1.20
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE3.png"></p>

```sh
dig www.me.local @192.168.1.20
```
<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE4.png"></p>

## DDNS
First of all 
