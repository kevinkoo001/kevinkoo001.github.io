---
author: kevinkoo001@gmail.com
comments: true
date: 2014-02-18 02:02:11+00:00
layout: post
link: http://dandylife.net/blog/archives/185
slug: bgp-security
title: BGP Security
wordpress_id: 185
categories:
- Attack &amp; Defense, Cyber Warfare
- Network Security
tags:
- BGP
- network
- Protocol
---

[You can download the slide: BGP Security](http://dandylife.net/blog/wp-content/uploads/2014/02/BGP-Security.pdf)
(This has been done as a part of homework in CSE508 in SBU CS.)

There are two different kinds of routing protocols: one is for interior purpose - IGP (Interior Gateway Protocol) and the other is for exterior purpose - EGP (Exterior Gateway Protocol). A good example of IGP would be RIP(Routing Information Protocol), OSPF (Open Shortest Path First) which is the most widely used, and EIGRP (Enhanced Interior Gateway Routing Protocol) which is proprietary by Cisco. In EGP, BGP (Border Gateway Protocol) is now de facto standard adapted by the Internet.

First we need to define a couple of terminologies.

1. AS (Autonomous System): A set of computers and routers under a single administration
2. RIB (Routing Information Base): BGP routing entries (Adj-RIB-In, Loc-RIB, and Adj-RIB-Out)
3. BGP Attribute: Types to decide path vector algorithm
(Origin=1, AS_Path=2, Next_Hop=3, MED=4, Local_Pref=5, Atomic_Aggregate=6, Aggregator=7)

As of Feb. 2014, there are more than 500,000 BGP routing tables available. (Check [http://bgp.potaroo.net/bgprpts/rva-index.html](http://bgp.potaroo.net/bgprpts/rva-index.html)) You may also want to know current AS summary. (Check [http://cidr-report.org/as2.0/](http://cidr-report.org/as2.0/)) Each BGP speaker uses RIBs and BGP attributes and installs NLRI (or best path) according to the following mechanism. (If the preference ties, then it considers next attribute in order.)

**Highest weight →Highest LOCAL-PREF → Originated Source → Shortest AS-PATH → Lowest Origin (IBGP < EBGP < incomplete) → Lowest MED → EBGP over IBGP → Lowest IGP Metric → Lowest Route ID → Lowest Originator ID**

The following figure illustrates 4 main BGP messages: OPEN, KEEP-ALIVE, UPDATE, and NOTIFICATION. The communication among BGP speakers maintains unicast over 179/tcp.

[![bgp1](http://dandylife.net/blog/wp-content/uploads/2014/02/bgp1.jpg)](http://dandylife.net/blog/wp-content/uploads/2014/02/bgp1.jpg)

Now, let's briefly take a look at BGP vulnerabilities from two perspectives. By running over TCP, listening on port 179, BGP is subject to be vulnerable through all kinds of TCP attacks: IP Spoofing, TCP RST,  TCP RST using ICMP, Session Hijacking, and various denial of service attacks including SYN flooding and so forth. These lead target router to drop the BGP session and both peers withdraw routes, causing disruption of network connection. An attacker takes advantage of eavesdropping, blackholing, and/or traffic analysis by changing routes as well.

[![bgp2](http://dandylife.net/blog/wp-content/uploads/2014/02/bgp2.jpg)](http://dandylife.net/blog/wp-content/uploads/2014/02/bgp2.jpg)

On top of that, with respect to BGP attacks, fundamental vulnerabilities arise from no mechanism which has specified within BGP in order to (a) validate the authority of an AS and (b) to ensure the authenticity of the path attribute by an AS. This allows an adversary to route manipulation such as message relaying, insertion, deletion, and modification as well as route hijacking. BGP-oriented attacks include:

(1) Route Flapping: repetitive changes rapidly cause the BGP routing table to be withdrawn and then re-advertised
(2) Route Deaggregation: announcing more specific route UPDATE causes a huge number of updates, which makes router crash and shut down
(3) (Unallocated) Route Injection: sending out incorrect routing information or transmitting routes to "bogon" prefixes

[![bgp3](http://dandylife.net/blog/wp-content/uploads/2014/02/bgp3.jpg)](http://dandylife.net/blog/wp-content/uploads/2014/02/bgp3.jpg)

Lastly, here are BGP attack countermeasures to mitigate corresponding threats above.

**1. Use authentication mechanism**



	
  * Use access control list.

	
  * Use BGP peer authentication: MD5(Routing Advertisement + Shared Key), IPSecif available

	
  * Configure BGP to allow announcing only designated netblocks

	
  * Disable BGP version negotiation to provide faster startup

	
  * Announce only preconfigured list of networks


**2. Configure route manipulation protection**



	
  * Use BGP graceful restart

	
  * Use max prefix limits to avoid filling router tables

	
  * Filter all bogonprefixes with ingress/egress filtering

	
  * Do not allow over-specific prefixes

	
  * Turn off fast external failover, called route flap damping

	
  * Record peer changes


**3. Use secure protocol**



	
  * Only allow peers to connect to port 179 in TCP

	
  * Randomize sequence number (against spoofing and session hijacking)

	
  * Consider deploying S-BGP or BGPSec


**[References]**

RFC 4271 -A Border Gateway Protocol 4 (BGP-4), which obsoletes RFC 1771, 1772
RFC 4272 -BGP Security Vulnerabilities Analysis
RFC 2439 –BGP Route Flap Damping
[http://moo.cmcl.cs.cmu.edu/~dwendlan/routing/](http://www.cisco.com/web/about/security/intelligence/protecting_bgp.html BGP Security)
[http://www.cisco.com/web/about/security/intelligence/protecting_bgp.html](http://www.cisco.com/web/about/security/intelligence/protecting_bgp.html)
