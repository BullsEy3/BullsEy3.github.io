---
title: SNMP
author: 
date: 2022-11-14
categories: [Protocols]
tags: [SNMP]
math: true
mermaid: true
image:
  path: https://i.postimg.cc/wjgXTrWD/screenshot-50.png
  width: 800
  height: 500
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

<h1> <b>SNMP</b> </h1>

<h3> About SNMP </h3>

The Simple Network Management Protocol or SNMP is among the least understood protocols, yet so vitally important to the successful operation of your network. If an attacker can breach the SNMP, they may be able to unmask your encrypted VPN communication, as well as see and possible control every device connected to your network .

The Simple Network Management Protocol uses UDP ports  161 and 162 to manage network devices. 
When used with Transport Layer Security or Datagram Transport Layer Security, requests are received on port 10161 and notifications are sent to port 10162.
Network devices use this protocol to communicate to each other and can be used by administrators to manage the devices. As hackers, if we can access the SNMP protocol, we can harvest a vast resource of information on the target's network and even disable and change the settings on these devices. Imagine the havoc one could wreak by changing the settings on routers and switches!

SNMP is a stateless, datagram oriented protocol. It involves one or more administrative computers called managers. These managers monitor and manage a group of computers. Each of the managed computers has an agent installed that communicates with the manager. Please see the diagram below  for  a schematic on how SNMP operates.

![image](https://i.postimg.cc/cHgN14hw/screenshot-7.png)

The agent on the managed computers provides management data to the managing computer. The manager can undertake management tasks, including modifying and applying new configurations.

The management data exposed by the agents on each of the managed machines is stored in a hierarchical database called the Management Information Bases or MIB. It is this information within the MIB that we will be seeking here. This MIB contains a vast array of information on every device on the network, including users, software installed, operating systems, open ports, etc. All of this information can be invaluable in developing an exploitation strategy on the target.

The SNMP protocol communicates on UDP port 161. The communication takes place with protocol data units or PDU's. These PDU's are of seven (7) types. 

    GetRequest  

    SetRequest  

    GetNextRequest  

    GetBulkRequest  

    Response  

    Trap  

    InformRequest 

 
 <b>SNMP Versions</b>

SNMP has three (3) versions. Version 1 or SNMPv1, has very poor security. The authentication of clients is in cleartext and by default, uses a "community string" that is set to "public". This community string operates like a password and it is valid for each and every node on the network. The authentication of the manager is also a community string set to "private",by default. With these community strings, the attacker can gather all the information from the MIB (with the public community string) and even, set the configuration on the devices (with the private community string). Although it is widely known and understood that SNMPv1 is insecure, it remains in wide use (I recently did a security assessment at a major NYC bank and they were still using SNMPv1). Even if the network administrator changes the community string from the defaults, because communication is in cleartext, an attacker can sniff the authentication strings off the wire.

SNMPv2 improved upon SNMPv1 in terms of performance and security, but because it was not backwardly compatible with SNMPv1, it was not widely adopted.  SNMPv3 is significantly more secure than either SNMPv1 or v2.  SNMPv3 adds encryption, message integrity and authentication, but is still not used on all networks.

https://www.youtube.com/watch?v=2IXP0TkwNJU

https://www.youtube.com/watch?v=qSot_GimBFk

https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp

https://www.netadmintools.com/snmp-mib-and-oids#wbounce-modal





<h2>Abusing SNMP for Information Gathering</h2>

- Open Kali and go to Applications --> Kali Linux -->Information Gathering --> SNMP Analysis -->snmpcheck

Snmpcheck is a Perl script that queries the SNMP MIB for information on the target IP. It's syntax is fairly simple;

``snmp-check``

Of course, some options are available such as the community string (it uses "public" by default), the SNMP version (it uses 1 by default or 2 is the other option. Note, it will not work on the more secure SNMP v3) and few others. We will be using it here against a 2003 Server on our network to see what information SNMP can provide us about the target.

<h2> Cracking SNMP community strings </h2>

In the previous exercise, we assumed that the admin had left the community string set to "public". What if the admin was a bit more cautious and security minded and had changed the community string? How can we find the community string?

There is an excellent tool built into Kali named onesixtyone (presumably named  after the default port that SNMP operates on).  In essence, it is a SNMP community string cracker. Like most "password" crackers, it relies upon a dictionary or wordlist to try against the service until it finds a match.

Let's open onesixtyone by going to Applications --> Kali Linux --> Information Gathering -->SNMP Analysis -->onesixtyone. It should open a help screen like below.

![image](https://i.postimg.cc/dtFbkK9M/screenshot-8.png)

The syntax of onesixtyone is pretty simple and  straightforward.

``kali > onesixtyone  [options] <host IP> <community string private or public>``

Like an dictionary-based password cracker, the dictionary you use with onesixtyone is  critical. In the case of onesixtyone, it has a built-in dictionary. It's small, but contains many of the commonly used strings with SNMP. If you are creating your own dictionary for SNMP cracking, this is a good starting point, but you may want to expand it with variations of the domain name or company name as network administrators don't usually put much effort in creating complex strings for SNMP.. For instance, if the company is Microsoft, you might try strings that a lazy admin might use, such as microsoft-public, microsoft-private, microsoft-snmp,  microsoft-network, etc. 

As you can see, it includes a short list of widely used SMNP community strings.

In this exercise, we will use this short and simple dictionary to see whether we can find that community string on our network and then use it in snmpcheck to gather all the info on the target.

`` kali > onesixtyone  192.168.1.102 -c /usr/share/doc/onesixtyone/dict.txt ``

![image](https://i.postimg.cc/7LPswHD9/screenshot-9.png)

As you can see in the screenshot above, it was able to find both  the  private community string (still set to the default "private") and the public community string (still set to the default as "public"). These community stings can then be used with snmpcheck to grab information from the MIB about the target system.

NSA Exploits SNMP to Unmask VPN Communications

We know that the NSA has exploited SNMP to unmask VPN communications from documents released by Edward Snowden. 

Conclusions :

SNMP can be a rich source of information on the target network if we can access it. snmpcheck will pull the information from the MIB and onesixtyone helps us crack the SNMP "passwords". Both can be critical in exploiting SNMP for reconnaissance.



