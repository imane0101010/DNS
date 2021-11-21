<h1 align="center">Domain Name System</h1>  

# Index
- #### [DNS Overview](https://github.com/imane0101010/DNS#dns-overview-1)
    * [Definition, Role and How it works](https://github.com/imane0101010/DNS#definition-role-and-how-dns-works)
    * [DNS servers]()
- #### [DNS configuration](https://github.com/imane0101010/DNS#dns-configuration-1)
    * [Server configuration](https://github.com/imane0101010/DNS#configuring-the-dns-server)
    * [Client configuration](https://github.com/imane0101010/DNS#configuring-the-dns-client)
    * [Primary and secondary DNS servers configuration](https://github.com/imane0101010/DNS#configuring-master-and-slave-servers)
- #### [DDNS configuration](https://github.com/imane0101010/DNS#ddns)
    * [DNS server configuration](https://github.com/imane0101010/DNS#dns-server-configuration)
    * [DHCP configuration](https://github.com/imane0101010/DNS#dhcp-server-configuration)
    * [Testing](https://github.com/imane0101010/DNS#testing-the-servers)  
  
# DNS Overview
## Definition, Role and How DNS works
The Internet - or any network for that matter - works by allocating a locally or globally unique IP address to every endpoint (host, server, router, interface, etc.). But without the ability to assign some corresponding name to each resource, every time we want to access a resource available on the network, it would be necessary to know its physical network address. With millions of hosts and resources, it’s an impossible task.  
To solve this problem, the concept of name servers was created in the mid-1970s to enable certain attributes (or properties) of a named resource, in this case, the IP address, to be maintained in a well-known location. The basic idea being that people find it much easier to remember the name of something, especially when that name is reasonably descriptive of function, content, or purpose, rather than a numeric address.  
The key concepts of the Internet’s Domain Name System are:
- **Authority** : being responsible for a particular node in the domain name hierarchy.  
- **delegation** : the process by which the authority at a higher level in the domain name hierarchy may transfer authority to lower levels.  
 
The iteration process of a translation of the name `abc.company.com` to an IP address is shown in the following figure below:  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/48b61bfa9d1db9f3b7dfeec572b67c6d1b59436b/DNS/Additional-files/translation-domain-name.PNG"></p>

## DNS servers
Name servers differ according to the way in which they save data: 
- **Primary name server/primary master** is the main data source for the zone. It is the authoritative server for the zone. This server acquires data about its zone from databases saved on a local disk.
- **Master name server** is an authoritative server for the zone. It is always published as an authoritative server for the domain in NS records. The master sever is a source of data of a zone for the subordinate servers (slave/secondary servers). There can be several master servers.
- **Secondary name server/slave name server** acquires data about the zone by copying the data from the primary name server (respectively from the master server) at regular time intervals. It makes no sense to edit these databases on the secondary name servers, although they are saved on the local server disk because they will be rewritten during further copying. This type of name server is also an authority for its zones.
- **Root name server** is an authoritative name server for the root domain (for the dot). Each root name server is a primary server, which differentiates it from other name servers.
- **Slave name server** transmits questions for a translation to other name servers, it does not perform any iteration itself.  

# DNS configuration
## Prerequisites
* Make sure the DNS server has a static ip  
* Update the repository index  

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
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/reverse.me.local.db
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

## Configuring the DNS client
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
We need to configure BIND on the master server (`ns1.me.local`) to enable zone transfer to our secondary server (`ns2.me.local`).
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
- `192.168.1.10` : the IP adress of the master server `ns1.me.local`  
- `192.168.1.20` : the IP adress of the slave server `ns2.me.local`  

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
- Static IP address : `192.168.1.20`
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

# DDNS
Dynamic DNS keeps DNS records (zone files) automatically up to date when an IP address changes. It is used in large networks that host internal services, and use their own internal DNS and DHCP servers as well as small companies and individuals when they want to publish a service on the Internet, and that service is hosted within an internal or home network, knowing that home networks typically use a *NAT* router to connect to the internet which means that devices located on the internal network aren’t accessible from the Internet.

## Generating the key
First of all let us generate the key for verification that will be used to secure the exchange of information between DHCP and DNS server.  
```sh
sudo rndc-confgen
```  
## Creating the file `ddns.key`  
Let us create the file ddns.key as follows:  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6d85b06dfa7c7076f7a38f7336e6eb4b5af3d5d4/DNS/DDNS/DDNS_key.png"></p>

## Copying the key into the correct locations
We should copy this file to `/etc/bind/` and `/etc/dhcp` and adjust the file permissions as follows: 
```sh
cp ddns.key /etc/bind/
cp ddns.key /etc/dhcp/
chown root:bind /etc/bind/ddns.key
chown root:root /etc/dhcp/ddns.key
chmod 640 /etc/bind/ddns.key
chmod 640 /etc/dhcp/ddns.key
```
## DNS Server configuration
### Updating the zones
The DNS server must be configured to allow updates for each zone that the DHCP server will be updating. We will need a key declaration for our key, and two zone declarations - one for the forward lookup zone and the other for the reverse lookup zone -. To do so modify the file `/etc/bind/named.conf.local` as follows:  
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/d934561b46ae864b043de3f87a61afa34638d963/DNS/DDNS/DDNS2.png"></p>
 
### Creating the zone files
*Forward zone*                 |  *Reverse zone*
:-------------------------:|:-------------------------:
![forward](https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS4.png)  |  ![Reverse](https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS5.png)   

### Creating symbolic links
Finally we need to create links from `/var/cache/bind` to the actual zone files in `/etc/bind`. This is because `/etc/bind` is not writeable for bind, but `/var/cache/bind` is.
```sh
cd /var/cache/bind
ln -s /etc/bind/db.me.org .
ln -s /etc/bind/db.192.168.1 .
```
## DHCP Server configuration
Here is the complete `dhcpd.conf` file with a basic configuration for the subnet `192.168.1.0/24`: 
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS6.png"></p>
  
### Restarting the servers
```sh
/etc/init.d/isc-dhcp-server restart
/etc/init.d/bind9 restart
```  
## Testing the servers
Now let's test the DDNS in a client machine using `nslookup` command, the result is shown below.
<p align="center"><img width="60%" src="https://github.com/imane0101010/DNS/blob/6503de3aac5cae9ab263cf80cacce01754c93204/DNS/DDNS/DDNS.png"></p>
