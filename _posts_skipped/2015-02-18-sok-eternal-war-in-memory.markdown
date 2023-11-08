---
author: kevinkoo001@gmail.com
comments: false
date: 2015-02-18 23:33:52+00:00
layout: post
link: http://dandylife.net/blog/archives/426
slug: sok-eternal-war-in-memory
title: '[Paper Review] SoK: Eternal War in Memory'
wordpress_id: 426
categories:
- Attack &amp; Defense, Cyber Warfare
- Papers in Security
---

<table style="width: 700px;" >
<tbody >
<tr >

<td width="81" >**Title**
</td>

<td colspan="3" width="700" >SoK: Eternal War in Memory [[Link](http://www.cs.berkeley.edu/~dawnsong/papers/Oakland13-SoK-CR.pdf)]
</td>
</tr>
<tr >

<td width="81" >**Author**
</td>

<td width="444" >Laszlo Szekeresy, Mathias Payerz, Tao Wei, Dawn Song
</td>

<td width="51" >**Email**
</td>

<td width="143" >
</td>
</tr>
<tr >

<td width="81" >**Publishing**
</td>

<td width="444" >SP '13 Proceedings of the 2013 IEEE Symposium on Security and Privacy
</td>

<td width="51" >**Year**
</td>

<td width="143" >2013
</td>
</tr>
<tr >

<td width="81" >**Abstract**
</td>

<td colspan="3" style="text-align: justify;" width="638" >Memory corruption bugs in software written in low-level languages like C or C++ are one of the oldest problems in computer security. The lack of safety in these languages allows attackers to alter the program’s behavior or take full control over it by hijacking its control flow. This problem has existed for more than 30 years and a vast number of potential solutions have been proposed, yet memory corruption attacks continue to pose a serious threat. Real world exploits show that all currently deployed protections can be defeated. This paper sheds light on the primary reasons for this by describing attacks that succeed on today’s systems. We systematize the current knowledge about various protection techniques by setting up a general model for memory corruption attacks. Using this model we show what policies can stop which attacks. The model identifies weaknesses of currently deployed techniques, as well as other proposed protections enforcing stricter policies. We analyze the reasons why protection mechanisms implementing stricter polices are not deployed. To achieve wide adoption, protection mechanisms must support a multitude of features and must satisfy a host of requirements. Especially important is performance, as experience shows that only solutions whose overhead is in reasonable bounds get deployed. A comparison of different enforceable policies helps designers of new protection mechanisms in finding the balance between effectiveness (security) and efficiency. We identify some open research problems, and provide suggestions on improving the adoption of newer techniques.
</td>
</tr>
<tr >

<td width="81" >**Summary**
</td>

<td colspan="3" width="638" >**1. Attack model demonstrating four exploit types and policies mitigating the attacks in different stages.**[![memorycorruption1](http://dandylife.net/blog/wp-content/uploads/2015/02/memorycorruption1.png)](http://dandylife.net/blog/wp-content/uploads/2015/02/memorycorruption1.png)

**2. The different protection techniques according to their policy and comapres the performance impact, deployment status (DEP), compatibility issues, and main attack vectors that circumvent the protection.**

[![memorycorruption2](http://dandylife.net/blog/wp-content/uploads/2015/02/memorycorruption2.png)](http://dandylife.net/blog/wp-content/uploads/2015/02/memorycorruption2.png)
</td>
</tr>
<tr >

<td width="81" >**Note**
</td>

<td colspan="3" width="638" >


This paper systematically clarifies how memory corruption attacks have been performed for the last 30 years at a glance although many kinds of countermeasures have been introduced to mitigate them.It also explains how memory corruption bugs are still hard to fix completely, and discusses a couple of approaches. It illustrates pros and cons of each defense mechanism including data integrity, data-flow integrity, code pointer integrity, and control flow integrity.




The authors introduced a set of defenses which hardens memory corruption attacks like stack cookies, data execution prevention (DEP), address space layout randomization (ASLR). However, they are not enough when attack vectors focus on return-oriented programming (ROP), information leaks, prevalent use of user scripting and just-in-time (JIT) compilation. Still the defense against attacks using ROP and JIT is active research area.



</td>
</tr>
</tbody>
</table>
