---
author: kevinkoo001@gmail.com
comments: false
date: 2016-01-16 19:52:51+00:00
layout: post
link: http://dandylife.net/blog/archives/551
slug: code-reuse-attacks-and-defenses
title: Code Reuse Attacks and Defenses
wordpress_id: 551
categories:
- Attack &amp; Defense, Cyber Warfare
tags:
- code reuse
- rop
---

### **1. Introduction**

Writing bug-free codes is quite challenging - almost infeasible - due to its high complexity and the limitation of extensive testing for both functionality and performance. In particular, memory corruption bugs have provided adversaries with a stepping stone that can lead to successful attacks. Although modern programming languages are designed to be safe against memory corruption vulnerabilities, many applications and systems still use the C and C++ programming languages, which support low level features.

  
During the last thirty years, various defense mechanisms have been proposed and deployed in order to eradicate code injection attacks, such as buffer overruns. Along with them, the offense side responds against existing enforcement mechanisms by introducing new evasion techniques. Despite the fact that memory corruption has been recognized as a well-known issue over several decades, many buggy programs still suffer from this classic vulnerability, ranked third [according to MITRE](https://www.sans.org/top25-software-errors/). Memory corruption has been studied for quite a long time as a research topic, but the problem of achieving a trade-off between effectiveness and efficiency (or between security and performance) remains open. After both DEP and ASLR have been deployed in modern operating systems by default, a large portion of attacks employ the code reuse technique and its variants to bypass existing defense mechanisms. The arms race between code reuse attacks and defenses is still an ongoing operation.

 

### **2. Brief History**

Historically, the Morris worm of 1988 was recorded as the first instance of a code injection attack that had impact on real servers, propagating through the Internet. Taking advantage of a buffer overflow vulnerability, it could crash running systems on purpose. Attacks using memory corruption started to become a mainstream threat after the disclosure of their operation by Aleph One in 1996. Another notorious attack was the Code Red worm, which infected 250,000 machines during 9 hours all over the world against Microsoft IIS Servers in 2001. Two years later, SQL Slammer slowed down the Internet by infecting 75,000 machines within merely 10 minutes. These worms all relied on the exploitation of buffer overrun vulnerabilities. _([Code-Red: a case study on the spread and victims of an Internet](https://www.caida.org/publications/papers/2002/codered/codered.pdf)_  
[_worm_ by D.Moore et al.](https://www.caida.org/publications/papers/2002/codered/codered.pdf))

[![](http://dandylife.net/blog/wp-content/uploads/2016/01/divert_control_flow-e1452969001684.png)](http://dandylife.net/blog/archives/551/divert_control_flow-2)

Traditional memory corruption exploits can be achieved by pointing to the injected code on the stack or heap which data resides in, followed by executing them. Based upon this observation in the past, the primary goal is to execute the arbitrarily injected code by erroneously dereferencing a pointer for control over a target to exploit ([i.e., stack overflow, heap overflow, format string, etc.](https://courses.cs.washington.edu/courses/cse484/14au/reading/low-level-security-by-example.pdf)). The figure above shows typical diversion of the control flow through pointer manipulation. Various techniques have been introduced including stack canaries, stack cookies, shadow stack, **ISR** (Instruction Set Randomization) and **DSR** (Data Space Randomization). Finally, defenders focused on constraining the permission of execution in a certain memory region. The model that any memory region cannot hold both write and execute permissions is simple but strong enough to trigger significant reduction of previous attacks.

  
However, memory page protection was defeated by a novel exploitation technique called _ret2libc_. (_[The Advanced Return-into-lib(c) Exploits ](http://phrack.org/issues/58/4.html)_[by Nergal](http://phrack.org/issues/58/4.html)) The idea was to reuse the existing code in a executable region in order to transfer control over the target program. Soon the limited function calls in _ret2libc_ made attackers devise more sophisticated ways to execute the arbitrary code they need, that is, _return-oriented programming_ or _**ROP**_. Return-oriented programming generalizes the previous idea so that multiple code chunks can be chained together to achieve an exploit, rather than re-using whole functions from shared libraries. Each small piece of code, known as a gadget, performs some simple operation, followed by a control transfer instruction that points to the next gadget to be executed. This technique is powerful enough to bypass existing non-executable memory countermeasures.

  
In response to code reuse attacks, a new defense mechanism, [ASLR (Address Space Layout Randomization), was introduced by the PaX Team](https://pax.grsecurity.net/docs/aslr.txt). It randomizes the address space within virtual memory every time a process is created so that attackers cannot predict the exact code location at runtime. This strategy has raised a considerable bar because the unpredictability of the required code’s location can eventually lead to failure of the exploit code. For these reasons, today major popular commodity operating systems have adopted both non-executable stack and ASLR protections by default to protect against memory corruption vulnerabilities. ([See a detailed description of the DEP feature from Microsoft](https://support.microsoft.com/en-us/kb/875352)) Modern CPU architectures also have started to support a special bit.

  
Unfortunately, this seemingly robust solution has been defeated again through the use of memory disclosure vulnerabilities. Once the location of memory could be revealed at runtime, traditional ROP would be available with ease even under full randomization. This means that ROP variants can survive from detection and prevention even under both DEP and ASLR environment. Whereas typical ROP makes use of a series of gadgets ending with ret instructions to transfer control, ROP variants employ indirect addresses such as _call_ and _jmp_ to evade advanced defenses. A variety of studies have been conducted to attempt to reach practical solution from largely two perspectives. 

  
The first category falls into **randomization**, which hinders the knowledge of the code layout. The second one focuses on** CFI, control flow integrity,** since the heart of ROP takes full advantage of following unintended control flows with indirect branches. The former approach includes binary instrumentation, compiler-based code regeneration, code transformation as well as address space randomization. The latter restricts indirect control transfers by runtime checking routines using CFG (control flow graphs) acquired in advance. To reduce runtime overhead while monitoring at runtime, tools like _kBouncer_ and _ROPecker_ suggested relaxed constraints with hardware components to effectively catch the moment a ROP attack begins.

  
Another new type of code reuse attacks leverages JIT engines to construct ROP payloads on the fly. Even though JIT ROP needs to run in a specified environment, it is classified as a high threat in that it enables the evasion of all state-of-the-art defense mechanisms that have been introduced lately. Recent demonstration illustrated that a shellcode was launched successfully through a JIT ROP exploit by feeding malicious JavaScript into a web browser. To remedy these attacks, a few tools resilient to JIT ROP have been suggested by disabling direct and indirect memory disclosure channels.

 

### **3. Evolution of Code Reuse Attacks**

**Systemization of New Attack Surface**

To overcome the limitation of injecting code, adversaries began to target the existing code which is already executable. The _ret2libc_ attack was an initial attempt to divert control flow to existing code. This made it possible to spawn a shell without code injection, only by jumping to the beginning of the desired function call in a OS-provided shared library and thereby bypassing writable XOR executable defenses. However, _ret2libc_ was quite restricted to apply for general exploits because attackers can only call preexisting functions.

 

[![his1](http://dandylife.net/blog/wp-content/uploads/2016/01/his1.png)](http://dandylife.net/blog/archives/551/his1)

  
Soon they developed a more flexible technique, return-oriented programming, which can be seen as a generalized form of _ret2libc_. The main objective is to divert the original control flow by only reusing existing code. This approach has remarkably distinctive feature in that code reuse attack adds a new path to the control flow graph (CFG) rather than adding a new node to the CFG in code injection. Instead of injecting arbitrary code, adversary needs to construct any code of her choice by reverse-engineering and static analysis of a given binary. By shifting attack surface, an exploit itself seemingly becomes more complicated but allows for more flexible and powerful attacks. However, this does not necessarily mean code injection has been ended. Conversely, in fact, the attack today often combines code reuse with code injection to execute malicious payload. For example, in Windows, an attacker can bypass existing mitigation and execute the injected code of her choice using _VirtualProtect()_ to configure the permission of a certain memory location.

  
In 2007, Shacham presented the introduction of how ROP actually works for the first time, titled “_The geometry of innocent flesh on the bone: ret2libc without function calls (on the x86)_”. The building blocks of ROP are short code sequences from functions. It relies on a) unaligned x86 instructions, b) extremely dense x86 instruction set architecture (ISA). The goal is to find sequences ending with ret or 0xC3. Many gadgets (i.e., 5,483 out of 975,626 in 1.13MB binaries) were found for essential operations such as load/store, control flow, arithmetic, logic, and system calls. It is proved to be feasible to find useful gadgets in existing code space, which allows an attacker to offer the execution of arbitrary codes without injecting a single code at all. 

Researchers have shown that ROP poses a severe threat in various platforms: SPARC, embedded systems, and even kernel. A framework has been suggested to automate architecture-independent gadget search. The algorithm is capable of locating a Turing-complete gadget set, while translating machine code into an intermediate language allows for minimal architecture-dependent adjustments. Schwartz et al. released Q, which helps to automatically generate ROP payloads for binaries. It shows how hardening exploits to bypass modern defense mechanism comes in handy. Mona is also another gadget generation tool to help ROP automation. Tran et al. demonstrate that the first form of _ret2libc_ can be indeed Turing complete technique, therefore identical in expressive power to ROP. The idea is that it combines existing libc functions to construct arbitrary computations. Furthermore, well-defined semantics of libc allows an adversary to maintain compatible attacks among different families of OSes.

 

**Hardening the Attack Vector inside W XOR X and ASLR**

Although it is obvious that the combination of **W XOR X** and **ASLR** severely impedes traditional memory corruption attacks including code reuse exploits, there are several drawbacks to defeat them. First off, relative distances between memory objects still remain intact in ASLR. Layouts of stack and library table remain the same as well. Secondly, it is still guessable by brute force attack in low entropy such as 32-bit machine. A previous research shows the maximum possible entropy allowed by the virtual memory space does not provide sufficient randomization against brute-force or de-randomization. It said the user mode ASLR on 32-bit architectures only left 16 bit of randomness. Third, research shows that ASLR has not been fully adopted yet. Only 2 out of 16 among popular third-party applications in Windows supported ASLR. Only 5% (66 out of 1,298) binaries in /usr/bin used ASLR in Ubuntu Linux. It appears that performance degradation makes it less preferable to be widely accepted. Payer found that the overhead and side-effects of PIE (Position Independent Executable) was up to 26% with an average of 10% due to the increased register pressure. Lastly, information leak thwarts ASLR by enabling the calculation of the base address at runtime. It is often said to be a vital requirement for successful ROP exploits, leading more advanced code reuse attacks in the long run. 

 

[![his2](http://dandylife.net/blog/wp-content/uploads/2016/01/his2.png)](http://dandylife.net/blog/archives/551/his2)

  
Taking advantage of shortcomings, the technique to bypass PaX ASLR protection has been issued for the first time. It shows that a pointer can be successfully modified to point to a nearby address, by overwriting the least significant byte or bytes of a pointer. Wei et al. demonstrated that de-randomization can be feasible by simply stuffing heap with large objects and multiple copies of attack payload with heap spraying (or JIT spraying) technique. JIT compilation often implements Dynamic Code Generation (DCG), a.k.a Runtime Code Generation. (i.e., _JavaScript, Flash, SilverLight_). It constructs an x86 instruction flow which might have completely different semantic when executing with a couple of byte offsets. Using this property, JIT begins to be misused by code reuse exploit later on.

 

**Information Leaks**

Information leaks in code reuse attacks refer to the disclosure of a memory address in the virtual address space of a given application at runtime. Once an attacker can obtain the address of specific area, this knowledge allows her to infer additional information that helps to mount a control flow hijacking attack. Today it is regarded as an essential element to mount a successful exploit even under full randomization.  
  
A _Pwn2Own_ winner, Dutch hacker, successfully demonstrated an exploit against a fully patched Internet Explorer 8 in 64-bit Windows 7. He bypassed ASLR with a heap overflow to get the address of a dll file in a browser and evaded DEP after an use-after-free vulnerability. After this, diverse vulnerabilities have been reported with the help of memory disclosure. demonstrated that an adversary is able to learn precise information about randomized memory layout of the kernel through timing side channel on modern operating system. They took advantage of the fact that shared resources between user and kernel space can be abused to construct a side channel to reveal memory hierarchy.

 

**Bypassing Cutting-Edge Defenses**

As various defenses mechanisms have been proposed, based on randomization and CFI, attackers have also demonstrated that new strategies can be followed to circumvent cutting-edge defenses. Checkoway et al. showed that ROP attacks can be feasible without _ret_ instructions. Instead they employed indirect flow instructions such as _call_ and _jmp_, that behave like a return. They mentioned that a new method would have negative implications against several defense proposals using detection of frequent returns or compiler modification. Similarly, an alternative attack paradigm, jump-oriented programming, has been introduced. It eliminates the reliance on the stack and _ret_ instructions including return-like ones like _pop+jmp_. In order to govern the execution control without _ret_, it uses the dispatcher gadget that determines which functional gadget would be invoked the next.

  
Late studies insist that ROP is still non-trivial by showing the circumvention against a series of state-of-the art defense mechanisms. Goktas et al. evaluated cutting-edge CFI techniques and exploited looser notion of CFI in particular by claiming two main issues: a) an ideal CFI is too expensive for deployment and b) it requires additional data like source code or debug information, which is often unavailable to commercial products. One example method was _**call-oriented programming (COP)**_ while chaining gadgets. Concurrently but independently, Carlini et al. successfully bypassed two modern defenses by violating assumptions in **_kBouncer_ **and **_ROPecker_**. They demonstrated that call-preceded gadgets are sufficient for exploitable payload. Moreover, the two assumptions based on ROP observation was incorrect thus bypassable as followings: a) all return instructions target call-preceded addresses, but using call-preceded gadgets can defeat it; b) ROP attacks are built of long sequences of short gadgets, but large no-operation gadgets could successfully circumvent it. In the same context, Goktas et al. pointed out that a heuristic-based policy with CFI can be thwarted by carefully-crafted-gadgets. It mainly relies on two threshold parameters: the length of the individual gadget (LG) and the length of gadget chain. (LC). Although the larger LG and the smaller LC is preferred to hinder attack success, setting too large LG and too small LC increases false positives. Both _kBouncer_ and _ROPecker_ picked the thresholds (LG and LC) as 20 and 8, and 6 and 11 based on heuristics respectively. However, it turns out finding the right size appears to be fairly challenging at the intersection of true positive and false positive. Bittau et al. discovered a novel way to remotely find ROP gadgets (Blind ROP or **BROP**), and to automatically construct an exploit against a specific platform (i.e., nginx, yaSSL and MySQL in 64-bit Linux).

 

### **4. Evolution of Code Reuse Defenses**

As code reuse attacks have become a popular exploitation technique to employ for memory corruption vulnerabilities, a great deal of research has been performed in both academia and industry. The techniques introduced in these studies fall into predominately two categories: a) randomization, and b) control flow integrity (CFI). The first group focuses on the predictability of the address space for an adversary. This observation has attempted to break the knowledge of code layout by introducing artificial diversity with the help of randomization technique. Another group has pinpointed that the apex of the code reuse attack has to benefit from corrupted control flow. This observation strives to restrict the use of indirect branches against control flow hijacking and thus to achieve control flow integrity. This section covers how two approaches have evolved to resolve the attack and what are their strengths and weaknesses.

 

[![his3](http://dandylife.net/blog/wp-content/uploads/2016/01/his3.png)](http://dandylife.net/blog/archives/551/his3)

 

**Randomization: Address Space**

The credit of the initial idea for randomization goes back in 2001 from PaX Team. They suggested the scheme, address space layout randomization or ASLR, whose main objective is randomize base addresses of memory segments at every invocation so that an attacker cannot predict the absolute addresses for exploit purpose.

  
Early research in address space randomization was explored by Bhatkar et al., which developed so-called address obfuscation that randomizes the location of data and code section. It also used code diversification in their implementation as well. 

  
Verified with reasonable performance in implementation, ASLR has started to be widely adopted by major modern operating systems. Linux kernel has implemented ASLR by default since the released version 2.6.12 in 2005, which affects both executables and shared libraries to randomize address space. Linux also offered the feature to enable a PIE (Position-Independent Executable) option in compile time so that a binary can be mapped into a virtual address with a random base at runtime even earlier than ASLR. Microsoft Windows has released Vista version at the form of opt-in ASLR for compatibility issues with old applications in early 2007. It allows for the randomization of locations including stack, heap, PEB (Process Environment Block) and TEB (Thread Environment Block). Windows 7 or later version supports ASLR by default. Microsoft has released the tool since 2009, the Enhanced Mitigation Experience Toolkit or EMET, to enforce randomization in case of statically-mapped DLLs even under ASLR-enabled environment. Followed by Windows, Apple also introduced Mac OS X Leopard 10.5 in late 2007, which supports randomization for system libraries. They extended ASLR implementation to all applications from Mac OS X Lion 10.7 and iOS 4.3 for mobile platform since 2011. As shown above, the goal of ASR technique aims to mitigate control-flow hijacking attacks by randomizing the location of code and data and thereby to hamper address predictability with high probability. However, due to several drawbacks like insufficient entropy, more fine-grained randomization technique needs to be brought up.

 

**Randomization: Code Diversification (Transformation)**

The fundamental distinction in code diversification technique from ASR is that it transforms the original code without breaking intended semantics of the program by rewriting a target binary. It tries to achieve fine-grained ASLR in various granularities: reordering/substitution within a basic block, function-level randomization, instruction-level randomization, and permutation of basic blocks. Note that binary rewriting often requires additional information to preserve the semantics such as source code for re-compilation, debugging symbols, relocation information and disassembly with control flow graph.

  
The first attempt of code transformation can be found in address obfuscation technique for user applications, which includes the aforementioned ASLR from a kernel level. To this end, a) it randomizes the base addresses of stack, heap, DLL, text and data segments, b) it permutes the order of variables and routines, and c) it introduces random gaps between objects such as stack frame padding between the base pointer and local variables, random padding between successive _malloc_ allocation requests and between variables, spaces within routines and so forth. This requires both an accurate control-flow graph and a full rewriting of all the routines for full feature-enabled implementation. Another mitigation tool has also been made assuming source code is available. The transformation covers static data, code, stack, heap and DLL. Even though its effectiveness and entropy was sufficient in practice, but the overhead was somewhat high (11% on average) and was inapplicable in general for the binaries away from revealing source code.

  
Chongkyung et al. proposed the binary rewriting tool, address space layout permutation (**ASLP**) [19]. It places the static code and data segments to a random location and performs function-level permutation up to 29 bits of randomness on a 32-bit architecture. However, ASLP does not support stack frame randomization thus it is vulnerable to a _ret2libc_ attack. Unless relocation information is available in a binary, it may require re-linking and re-compiling process which makes it less practical for most COTS products.

  
Another attempt of code transformation is **In-Place Code Randomization** or **IPR**. It introduces four randomization techniques: a) instruction substitution; b) intra basic block instruction reordering; c) instruction reordering with register preservation code; and d) register reassignment. IPR practically has no overhead because it uses exactly the identical instructions while transformation process. It also requires no source code or debug symbols, but it relies on the accuracy of disassembly to draw control flow graph in advance. As authors mentioned, it is difficult to decide if the remaining gadgets would be sufficient to construct useful gadgets. Additionally, it might not be applicable for software that performs self-checksumming or runtime code integrity checks. Behind some of drawbacks, this technique could break around 80% of gadgets on average with negligible overhead. Their evaluation shows that several known exploits were successfully broken by ROP code generating tools such as Q and Mona. 

 

Jason Hiser et al. suggested another approach, named Instruction Location Randomization or **ILR**, which attempted to break priori knowledge of ROP gadget locations. In a nutshell, it statically randomizes most instruction addresses and dynamically using control flow in a virtual machine (VM). To achieve this goal, ILR takes arbitrary binary as an input into disassembly engine and does both indirect branch target analysis and call site analysis. Then it rewrites the rules by reassembly engine, using them when instructions are being fetched inside ILR VM. Meanwhile, this technique leads to inevitably higher overhead (13% on average) with VM as well as space overhead (104MB of rule files on average). In addition, gadgets will be remained because there are indirect branch targets which cannot be moved. The security of VM itself has to be guaranteed to safely apply to this technique.

  
Binary stirring (**STIR**) has been proposed by Wartell et. al. Its high-level architecture includes static rewriting and stirring at load time. It transforms legacy x86 application binaries into self-randomizing instruction addresses without source code or debug information. Then STIR statically randomizes basic blocks in each invocation at load time. Although this method attains high effectiveness in removing gadgets (99.99%) with low overhead (1.6% on average), the authors had to duplicate the code section to avoid misinterpretation (i.e., converting code into data or vice versa) by design in a conservative way, which ends up with space overhead (73% on average). However, it focuses on the main module of the application rather than dynamic linked libraries in Windows or shared objects (SOs) in Linux. This results in the failure of abundant gadgets in those files. Moreover, STIR cannot protect control-flow hijacking when calling a legitimate computed jump target with corrupted arguments.

  
Lately Davi et al. implemented another rewriting solution to mitigate code reuse attacks, called **XIFER**. By comparing with previous works, it tries to balance out a tradeoff between functionality and efficiency by defining ten criteria and properties: effectiveness, entropy, randomization frequency, required information, coverage, performance and so forth. XIFER is a on-the-fly rewriter at the form of shared library applied the principle of code diversity. Its overhead with SPEC2006 experiments showed only 1.2% on average at runtime. However, this tool also relies on the accuracy of disassembly and building the reference graph.

 

**Control-Flow Integrity: Static checks**

As alluded above, static CFI checking confines execution flow within the boundary of allowed control paths, focusing on statically determining the valid targets (i.e., calls, function returns) A number of attempts have been made, applying relaxed policy for the integrity of indirect control transfers. 

  
Researchers have introduced the compiler to be able to eliminate gadgets. **Return-less kernel** has been developed on a LLVM-based prototype and used it to generate a return-less kernel in FreeBSD. **G-Free** removes gadgets from binaries at compile time by eliminating all unaligned free-branch instructions and by protecting the remaining aligned free-branch instructions. In particular, when generating an executable, it avoids ret instructions and various opcodes that can be used in gadgets. Extending the LLVM compiler, **Control-Flow Restrictor**, also has been implemented to add CFI enforcement during compilation time of a given application in Apple’s iOS. The shortcoming of compiler-based approaches is that it requires source code and it should be applied to all modules to be effective. (i.e., using a crafted compiler)

  
Another idea is to enforce CFI on binary programs without recompilation. Mingwei et al. presented a novel way to apply CFI to stripped binaries without source code, compiler support, debugging information or the presence of relocation. They developed robust techniques for disassembly, static analysis and transformation of a given binary, which can work even for complex COTS products against control flow hijacking. However, it cannot eliminate the _ret2libc_ attack since it follows the original control flow with no violation. Obfuscated code can’t be covered either because it is quite challenging to obtain reliable static disassembly. The runtime overhead was 4.29% on average, while the space overhead was 1.39 times larger than the original file size on average.

  
Zhang et al. proposed **CCFIR** (Compact Control Flow Integrity and Randomization) to address both performance and compatibility issues in practice. It uses both CFI and binary transformation technique. CCFIR collects all legitimate indirect control flow instructions and limits jumps to anywhere but known locations. The gathered-white-list facilitates random permutation to raise the bar against hijacking likelihood. Binary transformation depends on relocation tables only rather than source code or debug information. The legitimacy checks took reasonable overhead. (3.6% on average)

 

**Control Flow Integrity: Dynamic Checks**

Another viewpoint has focused on mitigating ROP exploits by monitoring program execution at runtime rather than at compile time or load time. An early research which applied dynamic CFI checking at runtime is **DROP**. It aims to detect malicious code build using ROP. By looking at ret instructions, it records the popped addresses and check if those are within libc and check out the length of each gadget and the maximum length of continuous candidate gadgets. Although the limitation is obvious since DROP cannot detect short sequence of gadgets (i.e., less than 3) and only detect the gadgets from libc ending with a ret instruction. The performance overhead under DROP was 5 times slower on average, up to 21 times, which made it impractical in use. **DynIMA** was proposed as a runtime monitoring architecture. It is the first attempt to prevent ROP with the help of Trusted Computing mechanism which verifies the integrity of executables in an operating system. However, it suffers from performance mostly during dynamic taint analysis that marks any untrusted data as tainted and traces the propagation of tainted data.

  
Another novel idea has been suggested with the concept of **locking**. Like mutex, the lock asserts the correctness of the control flow of the application by inserting a lock code. This code represents the lock by simply changing a certain value in memory. Only valid destination can unlock the corresponding lock. Otherwise the violation of control flow would be detected, leading to make the program aborted. 

 

Meanwhile, several coarse-grained CFI techniques have been introduced. Davi et al. presented another ROP detection tool, **ROPdefender** to defend against ROP. It uses dynamic binary instrumentation (DBI) to keep a shadow stack that is updated by instrumenting call and ret instructions. If ROPdefender detects a unexpected call-ret pair by comparison with the address placed on a shadow stack, it thwarts further execution. But one of shortcomings was that it did not protect any gadgets ending with indirect jmp or call instructions. Besides, additional instrumentation imposes significant overheads (2x on average) to hinder the applicability in practice.

 

**ROPGuard** focused on the observation that critical API functions would be invoked to launch successful attack. To be specific, for each critical function at runtime, it verifies if the return address is executable, the instruction at return address is preceded with a call instruction, and call instruction goes back to the current function. The idea was worth enough to protect many known ROP exploits in the wild. It has been integrated with Microsoft EMET tool. Kayaalp et. al proposed so-called BR or branch regulation. It enforces control flow rules at the function granularity against ROP exploit. Instead of constructing the whole CFG, BR checks only unintended branches. This property is efficient to common ROP and JOP attacks but it limits the protection against _ret2libc_ that follows control flow. The overhead was acceptable. (around 2% on average)

  
As coarse-grained CFI suffers from the limitation of protection against ROP as well as severe overhead, fine grained CFI has been introduced. To overcome performance degradation, state-of-the-art defense mechanisms started to adopt hardware-support feature such as branch tracing store (BTS) and Last Branch Record (LBR). Modern CPU supports a built-in performance monitoring unit (PMU) for the purpose of measuring its performance parameters such as instruction cycles, cache hits, cache misses and so on. PMU includes BTS, LBR, event filtering, conditional counting and so forth. Using hardware-support functionality compensates for the practical drawbacks of the original CFI perspective.

  
For the first time, **CFIMon** leverages the BTS mechanism to analyze runtime traces on-the-fly. It collects legitimate control transfers and detect violation of CFI accordingly. However, CFIMon might lead false positives and false negatives. To date, one of cutting-edge detection mechanism, **kBouncer** has been demonstrated by Pappas et al. It leverages LBR to keep track of branch history, which allows for transparent operation and minimum runtime overhead. (1% on average) Another similar approach has suggested **ROPecker** by Cheng et al, taking advantage of LBR as well [18]. Both kBouncer and ROPecker observed two significant thresholds at the moment of successful ROP attacks. One is gadget chain and the other one is gadget size. Heuristically, the former sets the maximum amount of gadget chain to 20 and the tolerable value of gadget size to 8 by default. The latter sets them to 6 and 11 respectively. The shorter gadget chain or the longer gadget size makes an adversary harder to construct exploitable payload in the end. Thus one can say ROPecker has more strict CFI policy than kBouncer. However, both defense mechanisms largely rely on the very limited size of the LBR stack, which holds only 16 records. The benefit of these solutions has been again defeated by relaxed assumptions and the aforementioned limitation. They pointed out heuristic itself cannot be the ultimate solution. They demonstrated many different evasion techniques and niche attack vectors such as NOP operation gadget, long gadget, and call-preceded gadget to nullify the state-of-the-art defense mechanisms. 

 

Another novel attempt was made to harden even ASLR-disabled binaries stripped off the relocation information during compilation. As executable files that do not carry relocation information cannot be loaded into the preferred base address in Windows, it is often seen that it is specified at link time. Under this environment, they monitor memory accesses and control flow transfers with page table manipulation at runtime. In other words, the technique relies on the reconstruction of missing relocation information by discovering appropriate relocatable offsets on the fly. The study sheds light on the feasibility to expand the protection towards legacy binary which does not take advantage of ASLR.

  
Lastly, Payer et al. presented **Lockdown**, that protects binary-only applications and libraries by rewriting them. During runtime, Lockdown adjusts the CFG by growing and shrinking its size as executing new code and unloading libraries respectively. It also employs a shadow stack to enforce the strict integrity of _ret_ instruction pointers, which only allows the return to target the actual caller. The performance overhead was 19% on average.
