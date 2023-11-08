---
author: kevinkoo001@gmail.com
comments: false
date: 2015-03-01 04:34:44+00:00
layout: post
link: http://dandylife.net/blog/archives/462
slug: paper-smashing-the-gadgets-hindering-return-oriented-programming-using-in-place-code-randomization
title: '[Paper] Smashing the Gadgets: Hindering Return-Oriented Programming Using
  In-Place Code Randomization'
wordpress_id: 462
categories:
- Attack &amp; Defense, Cyber Warfare
- Security in Society
---





<table style="width: 700px;" >
<tbody >
<tr >

<td width="81" >**Title**
</td>

<td colspan="3" width="638" >Smashing the Gadgets: Hindering Return-Oriented Programming Using In-Place Code Randomization
</td>
</tr>
<tr >

<td width="81" >**Author**
</td>

<td width="444" >Vasilis Pappas,  Michalis Polychronakis and Angelos D. Keromytis
</td>

<td width="51" >** From**
</td>

<td width="143" >Columbia University
</td>
</tr>
<tr >

<td width="81" >**Publishing**
</td>

<td width="444" >SP '12 Proceedings of the 2012 IEEE Symposium on Security and Privacy
</td>

<td width="51" >**Year**
</td>

<td width="143" >2012
</td>
</tr>
<tr >

<td width="81" >**Abstract**
</td>

<td colspan="3" width="638" >


The wide adoption of non-executable page protections in recent versions of popular operating systems has given rise to attacks that employ return-oriented programming (ROP) to achieve arbitrary code execution without the injection of any code. Existing defenses against ROP exploits either require source code or symbolic debugging information, or impose a significant runtime overhead, which limits their applicability for the protection of third-party applications. In this paper we present in-place code randomization, a practical mitigation technique against ROP attacks that can be applied directly on third-party software. Our method uses various narrow-scope code transformations that can be applied statically, without changing the location of basic blocks, allowing the safe randomization of stripped binaries even with partial disassembly coverage. These transformations effectively eliminate about 10%, and probabilistically break about 80% of the useful instruction sequences found in a large set of PE files. Since no additional code is inserted, in-place code randomization does not incur any measurable runtime overhead, enabling it to be easily used in tandem with existing exploit mitigations such as address space layout randomization. Our evaluation using publicly available ROP exploits and two ROP code generation toolkits demonstrates that our technique prevents the exploitation of the tested vulnerable Windows 7 applications, including Adobe Reader, as well as the automated construction of alternative ROP payloads that aim to circumvent in-place code randomization using solely any remaining unaffected instruction sequences.



</td>
</tr>
<tr >

<td width="81" >**Summary**
</td>

<td colspan="3" width="638" >


**1. Introduction**




ASLR/DEP techniques can be still evaded because parts of the address space in Windows do not change due to executables with fixed loaded address and/or shared libraries incompatible with ASLR. And Some exploits allows one to calculate the base address of a DLL either by brute-force or through a leaked pointer.







**2. Existing mitigations against ROP and their limitation**


<table >
<tbody >
<tr >

<td width="277" >Technique
</td>

<td width="346" >Limitation
</td>
</tr>
<tr >

<td width="277" >ASLR/DEP
</td>

<td width="346" >
</td>
</tr>
<tr >

<td width="277" >Compiler extensions
</td>

<td width="346" >
</td>
</tr>
<tr >

<td width="277" >Code randomization (permutation of function order)
</td>

<td width="346" rowspan="2" >Precise and complete extraction of all code and data which is only possible when debugging information is available.
</td>
</tr>
<tr >

<td width="277" >Control-flow integrity (binary instrumentation)
</td>
</tr>
<tr >

<td width="277" >Runtime solutions
</td>

<td width="346" >Significant runtime overhead
</td>
</tr>
</tbody>
</table>


**3. Novel and practical approach: In-place code randomization**








	
  * Case I: Atomic instruction substitution
Idea: obfuscation, metamorphism
Same computation can be done by a countless number of different instruction combination
Instructions of a gadget can be substituted by a functionally equivalent but different sequence of instructions

	
  * Case II: Reordering intra basic block
Idea: using dependence graph, code block can be reordered

	
  * Case III: Reordering register preservation code
Idea: As long as callee-saved registers (ebx, esi, edi, ebp) are restored in the right order, their actual order on the stack is irrelevant.

	
  * Case IV: Register Reassignment
Idea: two registers can be swapping during parallel, self-contained regions by drawing CFG (Control Flow Graph)










**4. Results**








	
  * Target: x86 PE executables – 5,235 PE files

	
  * On average, the applied transformations effectively eliminate about 10% gadgets

	
  * It also probabilistically break about 80% of the gadgets




</td>
</tr>
<tr >

<td width="81" >**Note**
</td>

<td colspan="3" style="text-align: justify;" width="638" >This paper presents an in-place code randomization technique against ROP attack. The idea is that it could break the gadgets with ease by limited number of transformation of binary, maintaining the size of the binary exactly the same. This is done with the help of disassembly using IDA Pro. The conservative randomization guarantees to keep the original code. Although the gadgets might be found in remaining text area, it would be quite practical and effective to make attackers hard in order to take advantage of gadget collection. The result shows that only 10% gadgets elimination could break 80% in total.
</td>
</tr>
</tbody>
</table>

