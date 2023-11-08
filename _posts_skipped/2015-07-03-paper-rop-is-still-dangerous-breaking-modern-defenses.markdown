---
author: kevinkoo001@gmail.com
comments: false
date: 2015-07-03 20:15:00+00:00
layout: post
link: http://dandylife.net/blog/archives/516
slug: paper-rop-is-still-dangerous-breaking-modern-defenses
title: '[Paper] ROP is still Dangerous: Breaking Modern Defenses'
wordpress_id: 516
categories:
- Papers in Security
tags:
- memory corruption
- rop
---

<table width="700" >
<tbody >
<tr >

<td width="81" >**Title**
</td>

<td colspan="3" width="638" >ROP is still Dangerous: Breaking Modern Defenses [[link](http://nicholas.carlini.com/papers/2014_usenix_ropattacks.pdf)]
</td>
</tr>
<tr >

<td width="81" >**Author**
</td>

<td width="444" >Nicholas Carlini and David Wagner from University of California, Berkeley
</td>

<td width="51" >**Contact**
</td>

<td width="143" >N/A
</td>
</tr>
<tr >

<td width="81" >**Publishing**
</td>

<td width="444" >23rd USENIX Security Symposium
</td>

<td width="51" >**Year**
</td>

<td width="143" >2014
</td>
</tr>
<tr >

<td width="81" >**Abstract**
</td>

<td colspan="3" width="638" >Return Oriented Programming (ROP) has become the ex- ploitation technique of choice for modern memory-safety vulnerability attacks. Recently, there have been multi- ple attempts at defenses to prevent ROP attacks. In this paper, we introduce three new attack methods that break many existing ROP defenses. Then we show howto break kBouncer and ROPecker, two recent low-overhead de- fenses that can be applied to legacy software on existing hardware. We examine several recent ROP attacks seen in the wild and demonstrate that our techniques successfully cloak them so they are not detected by these defenses. Our attacks apply to many CFI-based defenses which we argue are weaker than previously thought. Future defenses will need to take our attacks into account.
</td>
</tr>
<tr >

<td width="81" >**Summary**
</td>

<td colspan="3" width="638" >


**1. ****Careful observation on previous the-state-of-the-art defense mechanism**




(1) kBouncer:








	
  * Illegal returns: ROP always issues returns to non-call-preceded addresses

	
  * Long sequences of "short gadgets": payload built with long gadgets

	
  * LBR (Last Branch Record) is checked only when system calls are invoked







(2) ROPecker








	
  * Keep track of executable set (pages marked executable)

	
  * Check if there is a long chain of gadgets (threshold: normal max=10, ROP min=17)

	
  * Max (normal) < Detection <= Min (ROP)

	
  * Likewise kBouncer, LBR checking when invoking critical system calls







(3) Evaluation of two previous defense mechanism ()





	
  * Generic: supports all types of ROP gadgets

	
  * Efficient: minimal runtime overhead (1-4% at most)

	
  * Transparent: No source code, debugging symbols, compilers, binary instrumentation




**2. Key attack primitives**




(1) Call-preceded ROP gadgets: Both defenses check if gadget is non-call-preceded




(2) Evasion attack: Both defenses use a length-based classifier




(3) History Flushing: Both defenses keep only a limited amount of history inspection








	
  * 70KB of binary code was sufficient to mount full ROP attacks

	
  * According to previous study, call-preceded gadgets are usually 6%

	
  * Use a mixture of both short and long gadgets for evasion to defeat runtime monitoring

	
  * Use long gadget to hide history




**3. Implication for defenses (Lessons Learned)**




(1) Do not rely on limited amount of history
(2) Choosing call-preceded ROP (6%) is feasible (in >70K text)
(3) Classifying code as “gadget” vs “non-gadget” is challenging




(4) What is fundamental properties to determine ROP attacks? (Open question in Research)




</td>
</tr>
<tr >

<td width="81" >**Note**
</td>

<td colspan="3" width="638" >




This paper illustrates how even cutting-edge defense mechanism against ROP attack could be broken with carefully chosen gadgets from an attacker's perspective. The authors shows that even intuitively less potential gadgets from call-preceded ones are sufficient to mount a valid attack. Also, heuristic-based approach to classify gadgets VS non-gadgets can be evaded. History flushing and evasion technique make it possible to defeat all meticulous inspections as well. They remains a question that it is time to consider the fundamental attributes of ROP attack for further exploits.

I made a presentation with this paper in a security reading group at Stony Brook University. The material is available [here](http://dandylife.net/blog/?attachment_id=519).



</td>
</tr>
</tbody>
</table>
