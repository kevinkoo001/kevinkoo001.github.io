---
author: kevinkoo001@gmail.com
comments: false
date: 2015-02-02 08:57:17+00:00
layout: post
link: http://dandylife.net/blog/archives/367
slug: paper-review-a-look-back-at-security-problems-in-the-tcpip-protocol-suite
title: '[Paper Review] A Look Back at “Security Problems in the TCP/IP Protocol Suite”'
wordpress_id: 367
categories:
- Network Security
- Papers in Security
tags:
- network
- TCP/IP
---

<table >
<tbody >
<tr >

<td width="81" >**Title**
</td>

<td colspan="3" width="638" >A Look Back at “Security Problems in the TCP/IP Protocol Suite” [[Link](https://www.cs.columbia.edu/~smb/papers/acsac-ipext.pdf)]
</td>
</tr>
<tr >

<td width="81" >**Author**
</td>

<td width="447" >Steven M. Bellovin
</td>

<td width="48" >**Email**
</td>

<td width="144" >bellovin@acm.org
</td>
</tr>
<tr >

<td width="81" >**Publishing**
</td>

<td width="447" >20th Annual Computer Security Application Conference “classic paper” (ACSAC)AT&T Labs - Research
</td>

<td width="48" >**Year**
</td>

<td width="144" >2004
</td>
</tr>
<tr >

<td width="81" >**Abstract**
</td>

<td colspan="3" width="638" >About fifteen years ago, I wrote a paper on security problems in the TCP/IP protocol suite, In particular, I focused on protocol-level issues, rather than implementation flaws. It is instructive to look back at that paper, to see where my focus and my predictions were accurate, where I was wrong, and where dangers have yet to happen. This is a reprint of the original paper, with added commentary.
</td>
</tr>
<tr >

<td width="81" >**Summary**
</td>

<td colspan="3" width="638" >** Security issues & Defenses**
<table >
<tbody >
<tr >

<td width="109" >**Issues**
</td>

<td width="69" >**Sub**
</td>

<td width="197" >**Problems/Possible Attacks**
</td>

<td width="249" >**Defenses/Countermeasures**
</td>
</tr>
<tr >

<td colspan="2" width="178" >**TCP Sequence **
** Number Prediction**
</td>

<td width="197" >1. Makes “r” possibly execute malicious commands2. Generates queue overflows so that trusted client lost messages (DoS)3. Further session hijacking attack
</td>

<td width="249" >1. A cryptographic hash function to create a separate sequence number space for each  “connection”, a connection being defined per RFC791 as the unique 4-tuple <localhost, localport, remotehost, remoteport>.2. Random ISN generation à negative effects on the correctness of TCP in the presence of duplicate packets, the sum of a sequence

of random increments will have a normal distribution, which implies that the actual range

of the ISNs is quite small with Central limit theorem (CERT CA-2001-09)
</td>
</tr>
<tr >

<td width="109" rowspan="4" >**Routing Issues**
</td>

<td width="69" >**Source**
** Routing**
</td>

<td width="197" >1. Addr-based authentication
</td>

<td width="249" >1. Configure routers to reject external packets2. Use firewall,3. Reject src-routed packets at border routers
</td>
</tr>
<tr >

<td width="69" >**RIP Attack**
</td>

<td width="197" >1. No authentication allows an intruder to send bogus routing info, whose entries are visible widely2. AS 7007 attack3. Spammers hijack route, inject spam, and then withdraw the route.
</td>

<td width="249" >1. Filter out packets with bogus source. (Network ingress filtering)
</td>
</tr>
<tr >

<td width="69" >**EGP**
</td>

<td width="197" >1. Impersonates a second E/G for AS2. Claims reachability for some network where the real GW is down.
</td>

<td width="249" >1. Reasonably secure due to restricted topologies, but now BGP
</td>
</tr>
<tr >

<td width="69" >**ICMP**
</td>

<td width="197" >1. ARP Spoofing
</td>

<td width="249" >1. Includes plausible sequence number2. ICMP redirect disabled
</td>
</tr>
<tr >

<td colspan="2" width="178" >**“Authentication” Server**
</td>

<td width="197" >
</td>

<td width="249" >1. Do not use it
</td>
</tr>
<tr >

<td width="109" rowspan="6" >**Applications**
</td>

<td width="69" >**Finger**
</td>

<td width="197" >1. Displaying useful info. about users
</td>

<td width="249" >1. Firewall blocks the finger protocol
</td>
</tr>
<tr >

<td width="69" >**Email(POP)**
</td>

<td width="197" >
</td>

<td width="249" >1. Use encryption mode - SSL
</td>
</tr>
<tr >

<td width="69" >**DNS**
</td>

<td width="197" >1. DNS Sequence number attack2. Intercepts virtually all requests to translate names to IP addresses, and supply the address of a subverted  machine instead3. DNS Zone transfer (AXFR): no authentication on the request
</td>

<td width="249" >1. DNSSec provides digitally signed resource records
</td>
</tr>
<tr >

<td width="69" >**FTP**
</td>

<td width="197" >1. FTP authentication in plaintext2. Anonymous FTP – bounce attack
</td>

<td width="249" >1. Cryptographic protection for FTP
</td>
</tr>
<tr >

<td width="69" >**SNMP**
</td>

<td width="197" >1. No authentication reveals MIB
</td>

<td width="249" >1. Use _community string_ (simple plaintext pass)2. SNMPv3 defines user-based security model which provides cryptographic authentication
</td>
</tr>
<tr >

<td width="69" >**Remote Booting**
</td>

<td width="197" >1. RARP with TFTP2. BOOTP with TFTP3. Impersonate the server and send false DATA packets
</td>

<td width="249" >1. 4 byte random transaction id2. DHCP
</td>
</tr>
<tr >

<td colspan="2" width="178" >**Trivial Attacks**
</td>

<td width="197" >1. LAN vulnerable to eavesdropping (ARP poisoning, smurf attack)2. TFTP with no authentication3. Reserved Ports
</td>

<td width="249" >
</td>
</tr>
</tbody>
</table>
Comprehensive Defenses: Authentication, Encryption, Trusted Systems
</td>
</tr>
<tr >

<td width="81" >**Note**
</td>

<td colspan="3" width="638" >This paper covers a general idea of overall security problems in TCP/IP suite. It also provides a lot of references to see countermeasures of certain issues at a glance.This paper is based on what the author wrote original paper back in 1989. It reviews different types of protocol flaws itself and possible attacks from TCP sequence guessing, routing protocol issues to application layer protocols such as finger, POP3, SNMP, DNS, and FTP. Also he corrected wrong descriptions in previous paper if any as well as defense techniques and their limitations against such attacks.
</td>
</tr>
</tbody>
</table>
