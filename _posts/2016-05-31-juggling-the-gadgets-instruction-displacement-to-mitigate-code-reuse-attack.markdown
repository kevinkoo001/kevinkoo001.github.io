---
author: kevinkoo001@gmail.com
comments: false
date: 2016-05-31 05:58:56+00:00
layout: post
link: http://dandylife.net/blog/archives/606
slug: juggling-the-gadgets-instruction-displacement-to-mitigate-code-reuse-attack
title: 'Juggling the Gadgets: Instruction Displacement to Mitigate Code Reuse Attack'
wordpress_id: 606
categories:
- Attack &amp; Defense, Cyber Warfare
tags:
- gadget
- return oriented programming
- rop
---

**I. Background**

As modern OS has banned running arbitrary **code by injection** (i.e., a page in a virtual memory space cannot be set both executable and writable permission at the same time by default), **code reuse** attack has gained its popularity by taking advantage of the existing permission such as [return/jump/call]-oriented programming. (i.e., ROP attack)

The essence of the attack is that an adversary has the power of **predicting address space** and **diverting control flow.** Hence, two main approaches to defend against code reuse are either to break the knowledge of code layout with **randomization** or to restrict the use of the branches with **control flow integrity**.

**II. Overview of Instruction Displacement and Gadgets**

This work focuses on the former perspective, code diversification in particular. [One of previous works ](http://dandylife.net/blog/archives/477)introduces an In-Place Randomization (IPR) including instruction substitution, instruction reordering and/or register reassignment. The advantage of IPR is that it could be applied to stripped binaries thus practical for real applications with (theoretically) no overhead. It assumed both incomplete control flow graph and inaccurate disassembly from a binary that has been stripped off additional information - debugging symbols and source code - during compilation. However, it ended up with remaining gadgets (20%) that might be enough for the construction of a functional ROP payload.

The idea is to break more gadgets by **instruction displacement**. The goal of this technique is to maximize the gadget coverage. It might be thought of another way on top of IPR. However, displacement does not necessarily combine with it. Instruction displacement can be tied to any diversification technique with incomplete gadget coverage in order to increase it.

The following figure illustrates an example of what gadgets look like. (Here they are defined by looking ahead up to 5 instructions long from a '_ret_' instruction for the purpose of comparison with previous work.) 

[![intended_vs_unintended](http://dandylife.net/blog/wp-content/uploads/2016/05/intended_vs_unintended.png)](http://dandylife.net/blog/archives/606/intended_vs_unintended)

The dotted box represents pre-discovered gadgets. Assume that the process of gadget discovery is known thus we have the same power of obtaining gadgets with an adversary. The bold letters mean the first byte of each instruction. There are six different gadgets varying from 2 to 10 bytes in size. G1, G5, and G6 are **intended gadgets** because the starting byte of the first instruction is the same with the intended instructions; whereas, G2, G3, and G4 are **unintended gadgets** because the starting byte of the first instruction is different from the intended instructions. This shows that a lot of gadgets are **nested** in nature.

**III. High Level View**

A high level view of gadget displacement can be seen as following.

[![high_view](http://dandylife.net/blog/wp-content/uploads/2016/05/high_view.png)](http://dandylife.net/blog/archives/606/high_view)

First, we obtain pre-computed gadgets and displace them to a new section, named ._ropf_. (which has meant _rop-proof _area) Note that the unit of displacement should be within a basic block to maintain the semantic of the original program, which is the most important requirement.

In order to displace gadgets, intuitively a _jmp_ instruction is required that takes 5 bytes space; 1 byte for mnemonic and 4 bytes for a relative address. In other words, 5-byte-space in a single basic block is necessary for displacement. The remaining area is filled with _0xCC_ or INT 3 (interrupt 3). The INT **3** instruction is used by debuggers to temporarily replace an instruction to set a break point. Therefore any attempt to access to it would interrupt a program.

Another consideration is when the displaced area contains any branches and calls with relative addresses. All code references should be re-computed properly. Likewise, when the displaced area includes any absolute address in a relocation table (i.e., ._reloc_ section in a PE file), it has to be updated accordingly as well.

**IV. Displacement Strategy**

To achieve both efficiency and effectiveness for binary instrumentation, we set up the strategy like followings:

  * First, in general, jumping back and forth between two sections (._text_ and ._ropf_) is required. However, it is not necessary if the displaced region ends with either unconditional jump or return instruction because they know where to go back. For example, a _ret_ instruction would take whatever value on the stack to return.
  * It would be better to keep the number of displaced regions low for performance degradation. This can be resolved by choosing the largest gadget to include all nested gadgets within a basic block. It helps to break the gadgets whose sizes are less than 5 bytes.
  * For intended gadgets, it is simple enough to find the starting byte of the first intended instruction of the gadget and displace it into a ._ropf_.
  * For unintended gadgets, find the instruction all the way back in the same basic block for displacement. Otherwise, an attacker could also follow the inserted jump to make use of the existing gadgets.  
  * Finally, we shuffle the displaced instructions around in a ._ropf_ to avoid generating the same binary.

Putting all things together, the following algorithm summarizes the above. Per each gadget, it decides whether or not the gadget can be broken when IPR is not available.

[![disp_algo](http://dandylife.net/blog/wp-content/uploads/2016/05/disp_algo.png)](http://dandylife.net/blog/archives/606/disp_algo)

**V. Binary Instrumentation**

In this work, PE (portable executable),  a standard format in Windows, has been targeted in x86 machine. (You may find [here](http://dandylife.net/blog/archives/388) useful for more about PE) Briefly, PE consists of several section headers and corresponding data.

[![binary](http://dandylife.net/blog/wp-content/uploads/2016/05/binary.png)](http://dandylife.net/blog/archives/606/binary)

Above all, a new .ropf section header is appended at the end of existing section headers and a .ropf section that displaced code snippets reside in at the end of the binary. Next, all relocation entries are rebuilt in a relocation section. And some optional header fields including size of code and checksum should be adjusted accordingly. Other than those, all other area has to be preserved as they are. Displacing other sections may increase a complexity a lot during binary instrumentation.

[![reloc_table](http://dandylife.net/blog/wp-content/uploads/2016/05/reloc_table.png)](http://dandylife.net/blog/archives/606/reloc_table)

For a relocation table, it should be entirely reconstructed rather than appending new entries. This is because if original entries would be left, the inserted jump instruction can be overwritten during loading a binary into memory. A relocation table in a PE file consists of multiple relocation blocks. Each block starts with relative virtual address (RVA), block size and a series of entries of 2-byte value each. The first 4 bits represent type, and the last 12 bits represent offset.

For the example of the entry 0x304C in the figure above, 0x3000 means a relocation entry type and 0x4C is an offset from a RVA of the block, which means the absolute address in a virtual address 0x1C04C has to updated appropriately when loaded. Note that total number of all entries should be identical at all times.

**VI. Evaluation**

From almost 2,700 samples from Windows 7, 8.1 and benign applications, more than 13 million gadgets has been found in total. 6.37% gadgets are located in the unreachable regions (in reds below) mostly because of the failure of drawing control flow graph. The following plot illustrates an interesting the distribution of gadget kinds (small ones, unintended ones, and call-preceded ones) and that of broken gadgets. 

[![result](http://dandylife.net/blog/wp-content/uploads/2016/05/result.png)](http://dandylife.net/blog/archives/606/result)

The next Venn diagram shows how two different techniques are **complementary** at a glance. The figures in parenthesis are the ones except unreachable gadgets. Total coverage with IPR only was about 85% whereas it was about 90% with displacement only. Using both, it goes up to 97%. The unbroken gadgets ends up with 2.6%.

[![venn](http://dandylife.net/blog/wp-content/uploads/2016/05/venn.png)](http://dandylife.net/blog/archives/606/venn)

For the overhead tests, the industry standard _SPEC2006 _has been used. The performance overhead was around 0.36% on average. Having been some negative overheads, we performed statistical t-test done by establishing the null hypothesis that the means of CPU runtime overhead between original binaries and instrumented ones are the same. The result was that it rejects to fail the null hypothesis. In other words, there is no statically significant difference for negative overheads with 95% confidence interval.

**VII. Discussion and Limitation**

First of all, the number of displaceable gadgets still depends on the coverage of disassembly and CFG extraction. In addition, displacement technique requires at least 5 byte space to insert _jmp_ instruction.

Next, it cannot defend against _ret2libc_ and _JIT-ROP._  However, for return-to-libc, the real attack with actual APIs often requires code reuse for setting up parameters. The latest research shows the idea of JIT-ROP defense by making pages just executable only without readable. Since it constructs the gadgets on the fly after information leak, therefore, displacement technique can be leveraged to prevent JIT-ROP for fine-grained randomization.

Lastly, it cannot break the entry-point gadgets, which was less than 1%. 

**VIII. Final words**

You may find [the paper ](http://dandylife.net/blog/wp-content/uploads/2016/05/displacement.asiaccs16.pdf) and [the slide](http://dandylife.net/blog/wp-content/uploads/2016/05/Juggling-the-Gadgets.pdf) useful in details, which presented in ACM Asia Conference on Computer and Communications Security 2016 ([ASIACCS 2016](http://meeting.xidian.edu.cn/conference/AsiaCCS2016/program.html)). The experimental code is now publicly available at [this repository](https://github.com/kevinkoo001/ropf) at my Github. 
