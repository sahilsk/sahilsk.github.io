---
layout: post
title: Site-to-site Vpn Setup on Aws
modified:
categories: articles
excerpt:
tags: [ipsec, openswan, vpn, cross region vpc peering]
comments: true
share: true
image:
  feature:
date: 2017-01-06T15:00:05+05:30
---

How-to guide on setting up site-to-site vpn across regions.
----------
VPC peering allows you to peer VPC's as long as they are in the same region and have unique CIDR. But what if your VPC's are across regions. 

Lets say you want connectivity between servers running in two different region: Singapore and Mumbai. You may want to setup database slaves in different regions for disaster recovery or your application may want to reach client side for business requirement? And, being DevOpSec you also want the traffic flow to be fully encrypted as well.

Whatever may be the reason you want fully encrypted traffic flow to-and-fro vpc in one region to another region. This article is about setting up one such solution to this problem using IPSec.

IPSec comes into picture
----
IPSec is an Internet Engineering Task Force (IETF) standard suite of protocols that provides data authentication, integrity, and confidentiality as data is transferred between communication points across IP networks. Best part is IPSec provides data security at the IP packet level. IPSec emerged as a viable network security standard because enterprises wanted to ensure that data could be securely transmitted over the Internet. IPSec protects against possible security exposures by protecting data while in transit.


Ipsec-tools , openswan, strongswan, libreswan etc are few such implementations of IPSec Protocol.


IPSec emerged as a viable network security standard because enterprises wanted to ensure that data could be securely transmitted over the Internet. IPSec protects against possible security exposures by protecting data while in transit.



How are we doing?
---

Read through this [article](http://www.slashroot.in/linux-ipsec-site-site-vpnvirtual-private-network-configuration-using-openswan) before continuing from here.
I strongly recommend it. It'll help you understand configuration parameters better.


- Launch two servers one in each VPC in public subnet with new security group
- Install openswan on both of them
- configure openswan
- configure both VPCs route tables
- Test connectivity


For understanding let say we have following VPC's in our infrastructure

	
```	
region                  Private IP              Public IP        Subnet
-------------------------------------------------------------------------------

Mumbai                 172.19.1.132         52.66.100.2        172.19.0.0/16
singapore(stage)       172.27.7.141         54.155.125.80       172.27.0.0/16
singapore(dev)         172.25.135.54        52.67.12.236         172.25.0.0/16

```


Installing Openswan on CentOS 7
----------

Make sure you are launching a public server and accessible via internet. Thereafter, install openswan on them.

	yum install openswan lsof

Install it at both side servers in singapore stage vpc and mumbai vpc.


Configure openswan
--------

### Singapore Stage VPC:

Create configuration file and put obvious details. For more understanding consider 'left' as your source i.e currently loggedin server detail and 'right' is your destination side details


Create configuration file `stg-sg-to-mumbai.conf`

	cd /etc/ipsec.d
	cat > stage-sg-to-mumbai.conf
	conn stage-sg-to-mumbai
	  type=tunnel
	  authby=secret
	  left=%defaultroute
	  leftid=54.155.125.80
	  leftnexthop=%defaultroute
	  leftsubnet=172.27.0.0/16
	  right=52.66.100.2
	  rightsubnet=172.19.0.0/16
	  pfs=yes
	  auto=start


For authentication we'll be using Pre Shared Key.

Create secret file with format similar to shown below:
	
		leftPublicIp RightPublicIp: PSK <KEY GOES HERE>

Create `stage-sg-to-mumbai-secret.secrets`

		cat > stage-sg-to-mumbai-secret.secrets
		54.155.125.80 52.66.100.2: PSK "mySuperSecretGoesHere"


Restart swan and run verify

	service ipsec restart; tail -F /var/log/messages
	service ipsec status

To verify use following command:

	sudo ipsec verify

Next, disable check for source destination check. Go to `Action -> Networking -> Change Source/Dest. Check -> Disable`
![source-destination check](/images/src-dest-disable.png)  and disable it.
 ![disable prompt](/images/disablecheck.png)

Security group need to be tweaked too:

```
Type              Protocol       PortRange           Source                      Why
All traffi          All                All           172.19.0.0/16           ie. receive from mumbai vpc
All traffic         All                All           172.25.0.0/16           ie. receive fromm singapore dev vpc
All traffic         All                All           52.66.100.2/32        ie. receive from mumbai openswan server
SSH                 TCP                22            <myJumpHostIp>/32       ie. ssh access via jump host only
ALL ICMP            ALL                N/A            0.0.0.0/0              ie. for mysql master-slave replication to work
```



### Mumbai  VPC


Create configuration file `stg-mumbai-to-sg.conf`

	cd /etc/ipsec.d
	cat > stg-mumbai-to-sg.conf
	conn stg-mumbai-to-sg
	  type=tunnel
	  authby=secret
	  left=%defaultroute
	  leftid=52.66.100.2
	  leftnexthop=%defaultroute
	  leftsubnet=172.19.0.0/16
	  right=54.155.125.80
	  rightsubnet=172.27.0.0/16
	  pfs=yes
	  auto=start

Again create secret. Make sure both side openswan servers has same secret key. Otherwise authentication will fail.


	cat > stg-mumbai-to-sg.secrets
	52.66.100.2 54.155.125.80: PSK "mySuperSecretGoesHere"


Restart swan and run verify

	service ipsec restart; tail -F /var/log/messages
	service ipsec status

To verify use following command:

	sudo ipsec verify

Next, disable check for source destination check. Same way as we did in singapore region. Consult screenshots above.

Security group need to be tweaked too:

```
Type                Protocol            PortRange         Source                     Why
All traffic         All                All           172.19.0.0/16           ie. receive from mumbai vpc
All traffic         All                All           172.25.0.0/16           ie. receive fromm singapore dev vpc
All traffic         All                All           54.155.125.80/32        ie. receive from singapore openswan server
SSH                 TCP                22            <myJumpHostIp>/32       ie. ssh access via jump host only
ALL ICMP            ALL                N/A            0.0.0.0/0              ie. for mysql master-slave replication to work
```



#### oh wait.. we have two vpc running in singapore region: Stage and Dev. How do we connect Dev VPC also with mumbai VPC ?

We'll be required to launch one more openswan server running in dev VPC and make configuration at mumbai side vpc to receive traffic for this also.

#### Can't we use the same openswan public server that we've launched in stage VPC?
 No. because route tables are in different VPC. Route table won't let you select instance running in different VPC to route traffic to. Therefore, you cannot your routing rules there. Hence, a second server need to be launched in dev vpc too.


At mumbai side, since we have only one VPC we can use the same openswan server but with one more configuration to add this dev vpc network information. Also, we'll add one more route entry in the same route tables. Example entries are shown below.



### To have singapore dev vpc connectivity with mumbai VPC

- launch openswan server in public subnet of dev vpc using steps given above. Make sure you have a new security group here also.
- Add following configuration changes


#### Singapore Dev VPC

```
cd /etc/ipsec.d/
cat > dev-sg-to-mumbai.conf
conn dev-sg-to-mumbai
  type=tunnel
  authby=secret
  left=%defaultroute
  leftid=52.67.12.236
  leftnexthop=%defaultroute
  leftsubnet=172.25.0.0/16
  right=52.66.100.2
  rightsubnet=172.19.0.0/16
  pfs=yes
  auto=start
```

Don't forget to add secret file as well

	cat > dev-sg-to-mumbai.secrets
	52.67.12.236 52.66.100.2: PSK "mySuperSecretGoesHere"

Restart swan and run verify

	service ipsec restart; tail -F /var/log/messages
	service ipsec status

To verify use following command:

	sudo ipsec verify

Next, disable check for source destination check. Same way as we did in singapore region. Consult screenshots pasted before.


#### Mumbai VPC

Since we're using same VPC we can use the same openswan server and add just a new configuration there


	cd /etc/ipsec.d
	cat > dev-mumbai-to-sg.conf
	conn mumbai-to-ap
	  type=tunnel
	  authby=secret
	  left=%defaultroute
	  leftid=52.66.100.2
	  leftnexthop=%defaultroute
	  leftsubnet=172.19.0.0/16
	  right=52.67.12.236
	  rightsubnet=172.25.0.0/16
	  pfs=yes
	  auto=start

Add secret as well


	cat > dev-mumbai-to-sg.secrets
	52.66.100.2 52.67.12.236: PSK "mySuperSecretGoesHere"


Restart swan and run verify

	service ipsec restart; tail -F /var/log/messages
	service ipsec status

To verify use following command:

	sudo ipsec verify


Modify route table
-----

We need to modify vpc route tables so that we can route traffic from singapore vpc to mumbai vpc and vice versa.
Route table simply says packet with given `Destination`  should go via this `Target`. And 'Target' will send packet to its right owner.


Example:

### Mumbai VPC Route table

Here we're modifying mumbai vpc route table. We're telling it that packets with destination <172.25.0.0/16>(singapore dev vpc subnet) or <172.27.0.0/16>(singapore stage vpc subnet) should go via mumbai openswan server identified by instance id as shown in the screenshot. Do similar changes for all route tables whosever subnet you want connectivity to be setup.

![route table example config](/images/routetableexample.png)


Same changes are required at Singapore side stage VPC routebles.


Test Connectivity
---------

Lets say `Server-A-In-Sg` listening on port `XYZ` want to reach `Server-B-In-Mumbai` listening on port `PQR`. Server-A-In-Sg security group will allow port XYZ from Server-B-In-Mumbai private ip say, `172.19.x.y`. 

Similarly, Server-B-In-Mumbai security group should allow traffic for port PQR from Server-A-In-Sg say, `172.25.p.q` . 
That's all required. You dont' have to add openswan server ip also again here in security groups because they are already added in route table.
> PS: For mysql-slave replication to work do enable ICMP traffic also.



To test you may use telnet utility:

From Server-A-In-Sg server try reaching Mumbai side server:

	telnet 172.19.x.y XYZ

From Server-B-In-Mum server try reaching Singapore side server:
	
	telnet 172.25.p.q PQR


If you see conected message you are all up.


Conclusion
-------

Openswan configuration is very easy to understand and write. On AWS all firewall level settings are taken care by security grups and route table thereby making process more snappier.
One may have some concerns for production setup. Like how to make it highly available? There Linux-HA can be used with floating ip technique. But more on this, perhaps in next article.





