---
author: kevinkoo001@gmail.com
comments: false
date: 2015-02-10 01:30:45+00:00
layout: post
link: http://dandylife.net/blog/archives/378
slug: paper-review-tor-the-second-generation-onion-router
title: '[PAPER REVIEW] Tor: The Second-Generation Onion Router'
wordpress_id: 378
categories:
- Anonymity
- Papers in Security
---

<table style="width: 700px;" >
<tbody >
<tr >

<td width="81" >**Title**
</td>

<td colspan="3" width="638" >Tor: The Second-Generation Onion Router [[Link](http://www.onion-router.net/Publications/tor-design.pdf)]
</td>
</tr>
<tr >

<td width="81" >**Author**
</td>

<td width="444" >Roger Dingledine, Nick Mathewson and Paul Syverson
</td>

<td width="51" >**Email**
</td>

<td width="143" >arma@freehaven.net
</td>
</tr>
<tr >

<td width="81" >**Publishing**
</td>

<td width="444" >13th USENIX Security Symposium
</td>

<td width="51" >**Year**
</td>

<td width="143" >2004
</td>
</tr>
<tr >

<td width="81" >**Abstract**
</td>

<td colspan="3" width="638" >We present Tor, a circuit-based low-latency anonymous communication service. This second-generation Onion Routing system addresses limitations in the original design by adding perfect forward secrecy, congestion control, directory servers, integrity checking, configurable exit policies, and a practical design for location-hidden services via rendezvous points. Tor works on the real-world Internet, requires no special privileges or kernel modifications, requires little synchronization or coordination between nodes, and provides a reasonable tradeoff between anonymity, usability, and efficiency. We briefly describe our experiences with an international network of more than 30 nodes. We close with a list of open problems in anonymous communication.
</td>
</tr>
<tr >

<td width="81" >**Summary**
</td>

<td colspan="3" width="638" >**I. Overview**



	
  * Circuit based, fixed-size cells

	
  * Design: Perfect forward secrecy, Separation of “protocol cleaning” from anonymity, No mixing, padding, or traffic shaping, Many TCP streams can share one circuit, Leaky-pipe circuit topology, Congestion Control, Directory Servers, Variable exit policies, End-to-end integrity checking, Rendezvous points and hidden services


**2. Related Work**

Mix-Net, Babel, Mix-master, Mixminion, Anonymizer, Java Anon Proxy, PipeNet, ISDN mixes,    Tarzan, MorphMix, Crowds, Hordes, Herbivore, P5, Freedom, Cebolla, Anonymity Network, …

**3. ****Design goals and assumptions**



	
  * Goals: Deployability, Usability, Flexibility, Simple Design

	
  * Non-goals: Not peer-to-peer, Not secure against end-to-end attacks, No protocol normalization, Not steganographic


**4. The Tor Design**



	
  * Overlay network: each OR runs as a normal user-level process, and OP(Onion Proxy) to fetch directories and establish circuits.

	
  * Two keys: long-term identity key to sign TLS certificate and OR’s router descriptor, short-term onion key to decrypt requests

	
  * Cells:
[![tor1](http://dandylife.net/blog/wp-content/uploads/2015/02/tor1.jpg)](http://dandylife.net/blog/wp-content/uploads/2015/02/tor1.jpg)



	
  * Circuits and streams: Once a sender established a circuit, he can send relay cells
[![tor2](http://dandylife.net/blog/wp-content/uploads/2015/02/tor2.jpg)](http://dandylife.net/blog/wp-content/uploads/2015/02/tor2.jpg)

	
  * Brief Steps:
(a) Opening and closing streams
(b) Integrity checking on streams
(c) Rate limiting and fairness:
(d) Circuit-level throttling: packaging window, delivery window)
(e) Stream-level throttling: _relay sendme_ cells to implement end-to-end flow control


**5. Rendezvous Points and hidden services**



	
  * Goals: Access-control, Robustness, Smear-resistance, Application-transparency

	
  * Integration with user applications:
FQDN (_x.y.onion_ where x=authorization cookie, y=hash of the public key)

	
  * Procedures
Bob generates a long-term public key pair to identify his service
Bob chooses some introduction points, and advertises them on the lookup service
Bob builds a circuit to each of his introduction points, and tells them to wait for requests
Alice learns about Bob’s service out of band, retrieving the details from the lookup service
Alice chooses an OR as the rendezvous point (RP) for her connection to Bob’s service
Alice opens an anonymous stream to one of Bob’s introduction points
Alice gives it a message including her RP, rendezvous cookie, and then a DH handshake
The RP connects Alice’s circuit to Bob’s. (RP can’t recognize Alice, Bob, or the data)
Alice sends a relay begin cell along the circuit, arriving at Bob’s OP and Bob’s webserver.
An anonymous stream has been established, then Alice and Bob communicate as normal.


**6. Other design decisions**



	
  * Denial of Service
(a) CPU-consuming denial-of-service attacks: requiring clients to solve a puzzle while TLS handshaking or accepting _create_ cells
(b) Disrupting a single circuit or link breaks all streams passing along that part of the circuit (end-to-end TCP ACK protocol)

	
  * Exit-policies and abuse
(a) Attackers can harm the Tor network by implicating exit servers for their abuse. (This remains an arms race (unsolved))
(b) Mitigation: each onion router’s exit policy describes to which external addresses and ports the router will connect.

	
  * Directory Servers
(a) Partitioning attack to deceive a client about the router membership list, topology, or current network state
(b) Act as an HTTP server, so clients can fetch current network state and router lists, and so other ORs can upload state info.


**7. Attacks and Defenses**



	
  * Passive Attacks
<table >
<tbody >
<tr >

<td >

	
    * Observing user traffic patterns.

	
    * Observing user content.

	
    * Option distinguishability.

	
    * End-to-end timing correlation.

	
    * End-to-end size correlation.

	
    * Website fingerprinting.



</td>
</tr>
</tbody>
</table>


	
  * Active Attacks
<table >
<tbody >
<tr >

<td >

	
    * Compromise keys.

	
    * Iterated compromise.

	
    * Run a recipient.

	
    * Run an onion proxy.

	
    * DoS non-observed nodes.

	
    * Run a hostile OR.

	
    * Introduce timing into messages.

	
    * Tagging attacks.

	
    * Replace contents of unauthenticated protocols.

	
    * Replay attacks.

	
    * Smear attacks.

	
    * Distribute hostile code..



</td>
</tr>
</tbody>
</table>


	
  * Directory Attacks
<table >
<tbody >
<tr >

<td >

	
    * Destroy directory servers.

	
    * Subvert a directory server.

	
    * Subvert a majority of directory servers.

	
    * Encourage directory server dissent.

	
    * Trick the directory servers into listing a hostile OR.

	
    * Convince the directories that a malfunctioning OR is working.



</td>
</tr>
</tbody>
</table>


	
  * Attacks against R/P
<table >
<tbody >
<tr >

<td >

	
    * Make many introduction requests.

	
    * Attack an introduction point.

	
    * Compromise an introduction point.

	
    * Compromise a rendezvous point.



</td>
</tr>
</tbody>
</table>




</td>
</tr>
<tr >

<td width="81" >**Note**
</td>

<td colspan="3" width="638" >One of the best papers in terms of anonymity implementation. The tor is currently one of the largest deployed anonymous network over the world. Although there are a variety of conceptually anonymous networks and several ones in practice, tor is the most popular (as of now having millions of users) amongst them.This paper presents a circuit-based low-latency anonymous communication service. It addresses the limitations for the first tor implementation by adding a large number of features such as perfect forward secrecy, directory servers, integrity checking, configurable exit policies, and location-hidden services via rendezvous points.As authors mentioned, one of the biggest problem would be exit abuse, however, this might be remediated if end-to-end confidentiality have been used. One of side effects is to take advantage of high-level anonymity for the malicious purpose like spreading malware, exploit routes to avoid being kept track and/or IP laundry, which seems obvious now.
</td>
</tr>
</tbody>
</table>
