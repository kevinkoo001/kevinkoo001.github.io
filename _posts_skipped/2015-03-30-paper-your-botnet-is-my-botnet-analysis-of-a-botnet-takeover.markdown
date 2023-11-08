---
author: kevinkoo001@gmail.com
comments: false
date: 2015-03-30 22:36:02+00:00
layout: post
link: http://dandylife.net/blog/archives/501
slug: paper-your-botnet-is-my-botnet-analysis-of-a-botnet-takeover
title: '[Paper] Your Botnet is My Botnet: Analysis of a Botnet Takeover'
wordpress_id: 501
categories:
- Malware
- Security in Society
tags:
- botnet
- Malware
- paper
---

<table width="700" >
<tbody >
<tr >

<td width="81" >**Title**
</td>

<td colspan="3" width="638" >Your Botnet is My Botnet: Analysis of a Botnet Takeover [[link](https://seclab.cs.ucsb.edu/media/uploads/papers/torpig.pdf)]
</td>
</tr>
<tr >

<td width="81" >**Author**
</td>

<td width="444" >Brett Stone-Gross, Marco Cova, Lorenzo Cavallaro, Bob Gilbert, Martin Szydlowski, Richard Kemmerer, Christopher Kruegel, and Giovanni Vigna
</td>

<td width="51" >**From**
</td>

<td width="143" >UCSB
</td>
</tr>
<tr >

<td width="81" >**Publishing**
</td>

<td width="444" >CCS '09
</td>

<td width="51" >**Year**
</td>

<td width="143" >2009
</td>
</tr>
<tr >

<td width="81" >**Abstract**
</td>

<td colspan="3" width="638" >Botnets, networks of malware-infected machines that are controlled by an adversary, are the root cause of a large number of security problems on the Internet. A particularly sophisticated and insidious type of bot is Torpig, a malware program that is designed to harvest sensitive information (such as bank account and credit card data) from its victims. In this paper, we report on our efforts to take control of the Torpig botnet and study its operations for a period of ten days. During this time, we observed more than 180 thousand infections and recorded almost 70 GB of data that the bots collected. While botnets have been “hijacked” and studied previously, the Torpig botnet exhibits certain properties that make the analysis of the data particularly interesting. First, it is possible (with reasonable accuracy) to identify unique bot infections and relate that number to the more than 1.2 million IP addresses that contacted our command and control server. Second, the Torpig botnet is large, targets a variety of applications, and gathers a rich and diverse set of data from the infected victims. This data provides a new understanding of the type and amount of personal information that is stolen by botnets.
</td>
</tr>
<tr >

<td width="81" >**Summary**
</td>

<td colspan="3" width="638" >


**1. Introduction (Approach)**








	
  * Passive analysis of secondary effects caused by the activity of compromised machines

	
  * Active study for botnets (Torpig) via infiltration

	
  * Properties of Torpig:
a. transmits identifiers that permit us to distinguish between individual infections
b. harvests data from various applications and information from the infected victims















**2. _Torpig_ Infrastructure and background**





[![torpig_network](http://dandylife.net/blog/wp-content/uploads/2015/03/torpig_network.png)](http://dandylife.net/blog/wp-content/uploads/2015/03/torpig_network.png)




(1) Background








	
  * Distributed to victims as part of Mebroot which makes use of the evasion technique by MBR (Master Boot Record) manipulation

	
  * Infected through drive-by-download attacks (inclusion of HTML tags to request JavaScript)

	
  * Injected a DLL into "explorer.exe" by installer, then loads kernel driver (disk.sys)

	
  * Contacted C&C server to obtain malicious modules initially, where all communication were done with a sophisticated, custom encryption algorithm

	
  * Uploaded stolen data (i.e stored passwords and accounts) into C&C server periodically

	
  * Took advantage of phishing site which consists of HTML form to enter sensitive info







(2) Domain Flux








	
  * Each bot uses a DGA, Domain Generation Algorithm, with domain flux which generates a list of "rendezvous points" that could be used by botmasters to control their bots

	
  * Bots queried a certain domain mapped onto a set of IPs, changing frequently







(3) Taking control of the botnet





	
  * By registering .com / .net domains for 3 weeks

	
  * Sinkhole the traffic, ended up with gathering 8.7GB web log and 69GB pcap data




**3. Botnet analysis**




(1) Stolen Data








	
  * AVG Antivirus Free v2.9 (or AVG)

	
  * Lookout Security & Antivirus v6.9 (or Lookout)

	
  * Norton Mobile Security Lite v2.5.0.379 (Norton)

	
  * TrendMicro Mobile Security Personal Edition v2.0.0.1294 (TrendMicro)







(2) Botnet Size








	
  * Counting bots by nid: 180,835,  by submission header fields: 182,914 machines

	
  * 1,247,642 unique IP addresses


(3) Some statistics from the paper

[![bot_statistics](http://dandylife.net/blog/wp-content/uploads/2015/03/bot_statistics.png)](http://dandylife.net/blog/wp-content/uploads/2015/03/bot_statistics.png)












</td>
</tr>
<tr >

<td width="81" >**Note**
</td>

<td colspan="3" width="638" >It is a quite interesting paper because it contains live analysis of sensitive data harvested from the machines infected by real botnets on the fly. Also it is impressive to perform active infiltration impersonating C&C server in order to take over a botnet. This paper discusses one of the notorious botnets, Torpig, which was widely prevalent over the world back in 2009. The authors tried to reach comprehensive understanding on Torpig including infection path, static and dynamic analysis by reversing, relevant analysis of collected/stolen data from diverse perspectives.
</td>
</tr>
</tbody>
</table>
