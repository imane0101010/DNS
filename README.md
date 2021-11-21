<h1 align="center">Domain Name System</h1>  

# Index
- ## DNS configuration
    * Definition, Role and How it works
    * DNS servers
    * Server configuration
    * Client configuration
    * Primary and secondary DNS servers configuration
- ## DDNS configuration
    * DDNS server configuration
    * Client configuration
    * Testing 
  
# DNS configuration

## Prerequisites
make sure the DNS server has a static ip  
update the repository index  

## Installing Bind DNS Server
We will be using Bind DNS server, this collection of tools can be installed using the following command:  
```sh
sudo apt-get install -y bind9 bind9utils bind9-doc dnsutils
```  
## Configuring the DNS server
### Creating the zones
Let's begin by accessing the local DNS zone file `named.conf.local`.  
```sh
sudo nano /etc/bind/named.conf.local
```  
And then we will define the forward and the reverse zones for the `me.local` domain in the `named.conf.local` file as follows :  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-19%2014-53-38.png"></p>  

### Creating Zones Lookup files  
Once zones are created, we will create zone data files for the forward zone and reverse zone.  

#### Forward zone
```sh
sudo nano /etc/bind/forward.me.local.db
```  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-19%2014-53-08.png"></p>  

#### Reverse zone
```sh
sudo nano /etc/bind/reverse.me.local.db
```  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-19%2014-52-47.png"></p>  

### Check BIND configuration syntax
we will use two commands :  
- `named-checkconf` : checks the syntax of `named` configuration files for any errors.  
- `named-checkzone` : checks the syntax errors in zone files.  
```sh
sudo named-checkconf  
sudo named-checkzone me.local /etc/bind/forward.me.local.db
sudo named-checkzone 0.168.192.in-addr.arpa /etc/bind/reverse.me.local.db
```  
### Restarting bind
```sh
sudo systemctl restart bind9
```
### DNS status
```sh
sudo systemctl status bind9
```
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/Screenshot%20from%202021-11-17%2009-01-37.png"></p>  

## Configuration the DNS client
In the client machine, add the new DNS Server ip adress in the `/etc/resolv.conf` file:  
```sh
sudo nano /etc/resolv.conf
```
Make an entry like below (`192.168.1.10` is the server's IP).  
```sh
nameserver 192.168.1.10
```  
To verify our DNS Server,we can use the `dig` command to verify the reverse and forward lookup.
```sh
dig ns1.me.local
dig -x 192.168.1.10
```  
Forward lookup                   |  Reverse lookup
:-------------------------:|:-------------------------:
![forward](https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/dns_ans2.png)  |  ![Reverse](https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/DNS_CONFIGURATION/dns_ans1.png)   

## Configuring Master and Slave servers
### Master server
We need to configure BIND on the master server (ns1.me.local) to enable zone transfer to our secondary server (ns2.me.local).
#### Editing the zones
Edit the `/etc/named.conf.local` file in `ns1.me.local`.
```sh
sudo nano /etc/bind/named.conf.local
```  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/e3e86937d7119bd2b9c1f176a847a153aa1a406c/DNS/MASTER_&_SLAVE/MASTER/DNS_MASTER3.png"></p>

Note that we added the `allow-transfer` and `also-notify` parameters.
- `allow-transfer` allows zones transfer from the master to a slave server.  
- `also-notify` notifies a slave server when there has a change in zones at the master server.  
#### Editing the lookup files
- `192.168.0.10` : the IP adress of the master server `ns1.me.local`  
- `192.168.0.20` : the IP adress of the slave server `ns2.me.local`  

##### Forward zone
```sh
sudo nano /etc/bind/forward.me.local.db
```  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/MASTER_&_SLAVE/MASTER/DNS_MASTER.png"></p>

##### Reverse zone
```sh
sudo nano /etc/bind/reverse.me.local.db
```  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6947d3eb1706c5934ae3a6ae793950e23bc2ef6d/DNS/MASTER_&_SLAVE/MASTER/DNS_MASTER1.png"></p>

### Slave server
#### Prerequisites
- Static IP address : `192.168.0.20`
- Bind9 installed  

#### Editing the zones
Edit `/etc/bind/named.conf.local` file as shown in the following image.  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE1.png"></p>

- `masters` : allows to declare a dns server as slave when added to the zones by defining the IP adress of the master server.  
#### Restart Bind9
```sh
sudo systemctl restart bind9
```  
### Verification
Let us see if the forward and reverse zones file is already in `/var/cache/bind`  
```sh
sudo ls -l /var/cache/bind
```  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE2.png"></p>

Then let's add the DNS slave and master IP addresses in `/etc/resolv.conf`.
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE.png"></p>

Now, let's check the configuration using dig command
```sh
dig ns2.me.local @192.168.1.20
```
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE3.png"></p>

```sh
dig www.me.local @192.168.1.20
```
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/5fc76d4ee53b88b3e6e17d48108ec796e0c7bfdd/DNS/MASTER_&_SLAVE/SLAVE/DNS_SLAVE4.png"></p>

## DDNS
#### Generating key
First of all let us generate the key for verification that will be used to secure the exchange of information between DHCP and DNS server.

```sh
sudo rndc-confgen
```
#### Creating the file ddns.key
Let us create the file ddns.key as follows:

<p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6d85b06dfa7c7076f7a38f7336e6eb4b5af3d5d4/DNS/DDNS/DDNS_key.png"></p>

#### Copying the key into the correct locations

We should copy this file to /etc/bind/ and /etc/dhcp and adjust the file permissions as follows: 

```sh
cp ddns.key /etc/bind/
cp ddns.key /etc/dhcp/
chown root:bind /etc/bind/ddns.key
chown root:root /etc/dhcp/ddns.key
chmod 640 /etc/bind/ddns.key
chmod 640 /etc/dhcp/ddns.key
```
#### DNS Server configuration

 ##### Update of zones
 
 The DNS server must be configured to allow updates for each zone that the DHCP server will be updating. We will need a key declaration for our key, and two zone declarations - one for the forward lookup zone and one for the reverse lookup zone. To do so modify the file /etc/bind/named.conf.local as follows: 
 
 <p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/d934561b46ae864b043de3f87a61afa34638d963/DNS/DDNS/DDNS2.png"></p>
 
 ##### Creating the zone files
 
 ###### Forward Zone
 
  <p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS4.png"></p>
  
 ###### Reverse Zone
 
   <p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS5.png"></p>
   
 ##### Creating symbolic links
 
 Finally we need to create links from /var/cache/bind to the actual zone files in /etc/bind. This is because /etc/bind is not writeable for bind, but /var/cache/bind is.
 
```sh
cd /var/cache/bind
ln -s /etc/bind/db.me.org .
ln -s /etc/bind/db.192.168.1 .
```
##### DHCP Server configuration

Here is the complete dhcpd.conf file  with a basic configuration for the subnet 192.168.1.0/24: 
  <p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS6.png"></p>
  
##### Restarting the servers

```sh
/etc/init.d/isc-dhcp-server restart
/etc/init.d/bind9 restart
```
##### Testing the servers

Now let´s test the DDNS in a client machine using nslookup command.
 <p align="center"><img width="50%" src="https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS.png"></p>
  
