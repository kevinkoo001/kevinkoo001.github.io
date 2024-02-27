---
author: kevinkoo001@gmail.com
comments: true
date: 2018-05-23 22:31:16+00:00
layout: post
link: http://dandylife.net/blog/archives/743
slug: compiler-assisted-code-randomization
title: Compiler-assisted Code Randomization
wordpress_id: 743
categories:
- Attack &amp; Defense, Cyber Warfare
tags:
- return oriented programming
- rop
- software diversity
---

<table cellpadding="0" cellspacing="0" border="0" ><tbody ><tr style="height: 145px;" >
<td style="height: 145px;" >I. Motivation  
II. Compiler-assisted Code Randomization (CCR) Overview  
III. Identifying Essential Information for Randomization  
IV. Obtaining Metadata from the LLVM Backend  
V. Metadata Definition with Google’s Protocol Buffers  
VI. Consolidating Metadata in the gold Linker  
VII. Randomizer   
VIII. Evaluation
</td></tr></tbody></table>

#### **I. Motivation**

* * *

_**Code randomization**_ is not a new technique in the software security field. Rather, it is a well-known _defense against code reuse attack_ (a.k.a [return-oriented programming or ROP](https://en.wikipedia.org/wiki/Return-oriented_programming)) by breaking the assumption of the attackers that useful gadgets (arbitrary instruction snippets to choose from a potentially vulnerable process memory space) are available. One might argue the application with a scripting-enabled feature allows an adversary to have the power to scan the code segments on the fly (i.e., by leveraging a memory disclosure), rendering the defense invalid. Such a _just-in-time ROP_ (dubbed _JIT-ROP_) could be prevented with the _execute-only_ memory (_XOM_) concept by restriction of reading the code, which blocks the on-the-fly discovery of gadgets. Still the protection scheme is viable only upon diversified code as XOM would be pointless with the identical code page. (Note that here I use the following terms interchangeably: randomization, diversification, transformation, and shuffling.)

However, modern operating systems have only adopted _address space layout randomization_ (a.k.a _ASLR_) despite decades of research that shows _**software diversification** _is an effective protection mechanism. If so, why would software vendors not offer the protection? I believe it is presumably because it seems non-trivial to generate a unique instance at each end.  Having explored previous works on binary transformation, I found two major requirements for widespread adoption: i) _**reliable binary rewriting**_ (one would not desire a broken executable to boost security) and ii) _**reasonable cost**_ (creating hardened variants, distributing them and maintaining compatibility should not be costly). 

<blockquote>Code Randomization is an effective protection scheme against code-reuse attack. However, it has not been deployed by software vendors mainly due to the difficulty of rewriting binary reliably at a reasonable cost. Can we tackle the hurdle if the pre-defined transformation-assisting information could be obtained from a compiler toolchain?</blockquote>

For (i), one observation is that traditional binary rewriting requires complicated binary analysis and/or disassembly process at all times to create a sane executable.  Although **_static binary analysis_** (without running binary) and **_dynamic binary analysis_** have their own advantages, even using both might be sometimes far from the ground truth. Likewise, heuristics have limitation to obtain a reliable binary with full accuracy. For example, it is quite hard to identify function boundary correctly when the code has been highly optimized (i.e., -O3) by compiler. For (ii), it is obvious that the vendors would be reluctant to create numerous copies (i.e., a popular application like a browser used by tens of millions of end users) and to store them through the current distribution channel (i.e., CDN; Content Delivery Network). It could be even worse when those variants become incompatible with patching, crash reporting and other mechanisms on software uniformity.

The aforementioned hurdles had motivated me to hack a _toolchain_ itself that is responsible for building the final executable. In other words, _compiler and linker should be able to explain every single byte to be emitted_, which eventually guarantees the perfect executable (all the time) to preserve the original semantics without running it. Hence, it would be feasible to rewrite a reliable variant without a cumbersome analysis _**if only if**_ **_essential information for transformation could be extracted during compilation_**, which tackles the issue (i). The collected information allows one to generate his/her own instance **_on demand_ _at installation time_** once the final executable contains the information, which resolves the issue (ii).

#### **II. Compiler-assisted Code Randomization (CCR) Overview**

* * *

Here introduces_** Compiler-assisted Code Randomization (CCR),**_ a hybrid method to enable practical and generic code transformation, which relies on _compiler-rewriter cooperation_. The approach allows end users to facilitate rapid and reliable fine-grained code randomization (at both a function level and a basic block level) on demand at installation time. The main concept behind CCR is _**to augment the final executable with a minimal (pre-defined) set of transformation-assisting metadata**_. Note that the _LLVM_ and _gold_ have been chosen as compiler and linker for CCR prototype. The following table briefly shows the essential information that could be collected/adjusted at compilation/linking time. [table id=3 /]

The following figure illustrates the overview of the proposed approach from a high level at a glance. Once a modified compiler (LLVM) collects metadata for each object file upon given source code, the modified linker (gold) consolidates/updated the metadata and store it to the final executable. Once software vendor is ready for distributing a master binary, it is transferred to end users over legacy channel. A binary rewriter leverages the embedded metadata to produce the diversified instances of the executable. [![](http://dandylife.net/blog/wp-content/uploads/2018/05/overview-1.png)](http://dandylife.net/blog/?attachment_id=888)

<blockquote>For interested readers, you may jump into my [git repository](https://github.com/kevinkoo001/CCR) to play with CCR before moving on. The README describes how to build it and how to instrument a binary in detail. Here is the paper: _[Compiler-assisted Code Randomization](http://dandylife.net/blog/wp-content/uploads/2018/05/ccr.sp18.pdf) in [Proceedings of the 39th IEEE Symposium on Security & Privacy (S&P)](https://www.computer.org/csdl/proceedings/sp/2018/4353/00/index.html)_. The slides are available [here](http://dandylife.net/blog/wp-content/uploads/2018/05/CCR_final.pdf). 
> 
> </blockquote>

#### **III. Identifying Essential Information for Randomization**

* * *

This chapter does not explain extensive metadata described above. Instead it will guide how to identify the crucial information for transformation. Let us investigate how the LLVM backend emits the final bytes with a very simple example. The following source code just calls a single function and exit.
    
    // foo.c example
    #include <stdio.h>
    
    void foo(void);
    
    int main(void) {
            foo();
            return 0;
    }
    
    void foo() {
            printf("foo called!\n");
    }

The LLVM itself (compiled with a debugging option -g) allows us to use fine grained debug information with _DEBUG_TYPE_ and _-debug-only_ option ([See the programmers manual in the LLVM](http://llvm.org/docs/ProgrammersManual.html#the-debug-macro-and-debug-option)). Using the following compilation options, you could see what's happening inside the LLVM backend.
    
    $ clang3 -o foo -mllvm -debug-only=mc-dump foo.c
    ... (omitted)
    assembler backend - final-layout
    --
    <MCAssembler
      Sections:[
        <MCSection Fragments:[
          <MCAlignFragment<MCFragment 0xac7b7f0 LayoutOrder:0 Offset:0 HasInstructions:0 BundlePadding:0> (emit nops)
            Alignment:4 Value:0 ValueSize:1 MaxBytesToEmit:4>>,
          <MCAlignFragment<MCFragment 0xaca6b50 LayoutOrder:1 Offset:0 HasInstructions:0 BundlePadding:0> (emit nops)
            Alignment:16 Value:0 ValueSize:1 MaxBytesToEmit:16>>,
          <MCDataFragment<MCFragment 0xaca6e50 LayoutOrder:2 Offset:0 HasInstructions:1 BundlePadding:0>
            Contents:[55,48,89,E5,48,83,EC,10,C7,45,FC,00,00,00,00,E8,00,00,00,00,31,C0,48,83,C4,10,5D,C3] (28 bytes),
            Fixups:[<MCFixup Offset:16 Value:foo-4 Kind:6>]>,
          <MCAlignFragment<MCFragment 0xac938c0 LayoutOrder:3 Offset:28 HasInstructions:0 BundlePadding:0> (emit nops)
            Alignment:16 Value:0 ValueSize:1 MaxBytesToEmit:16>>,
          <MCDataFragment<MCFragment 0xaca8be0 LayoutOrder:4 Offset:32 HasInstructions:1 BundlePadding:0>
            Contents:[55,48,89,E5,48,83,EC,10,48,BF,00,00,00,00,00,00,00,00,B0,00,E8,00,00,00,00,89,45,FC,48,83,C4,10,5D,C3] (34 bytes),
            Fixups:[<MCFixup Offset:10 Value:.L.str Kind:3>,
                    <MCFixup Offset:21 Value:printf-4 Kind:6>]>]>,
        <MCSection Fragments:[
          <MCDataFragment<MCFragment 0xac92a60 LayoutOrder:0 Offset:0 HasInstructions:0 BundlePadding:0>
            Contents:[66,6F,6F,20,63,61,6C,6C,65,64,21,0A,00] (13 bytes)>]>,
        <MCSection Fragments:[
          <MCDataFragment<MCFragment 0xac92650 LayoutOrder:0 Offset:0 HasInstructions:0 BundlePadding:0>
            Contents:[00,63,6C,61,6E,67,20,76,65,72,73,69,6F,6E,20,33,2E,39,2E,30,20,28,74,61,67,73,2F,52,45,4C,45,41,53,45,5F,33,39,30,2F,66,69,6E,61,6C,29,00] (46 bytes)>]>,
    ... (omitted)

The output shows the final layout determined by the LLVM assembler. At first, the jargon seems unfamiliar but we know what the sections are in a ELF format. The line 7, 21 and 24 tell us the beginning of the section. Now let's see the section headers with a readelf utility.
    
    $ readelf -SW foo
    There are 32 section headers, starting at offset 0x1a80:
    
    Section Headers:
      [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
      [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
      [ 1] .interp           PROGBITS        0000000000400238 000238 00001c 00   A  0   0  1
      [ 2] .note.ABI-tag     NOTE            0000000000400254 000254 000020 00   A  0   0  4
      [ 3] .dynsym           DYNSYM          0000000000400278 000278 0000a8 18   A  4   1  8
      [ 4] .dynstr           STRTAB          0000000000400320 000320 000089 00   A  0   0  1
      [ 5] .gnu.hash         GNU_HASH        00000000004003b0 0003b0 00001c 00   A  3   0  8
      [ 6] .gnu.version      VERSYM          00000000004003cc 0003cc 00000e 02   A  3   0  2
      [ 7] .gnu.version_r    VERNEED         00000000004003dc 0003dc 000020 00   A  4   1  4
      [ 8] .rela.dyn         RELA            0000000000400400 000400 000018 18   A  3   0  8
      [ 9] .rela.plt         RELA            0000000000400418 000418 000048 18  AI  3  22  8
      [10] .init             PROGBITS        0000000000400460 000460 00001a 00  AX  0   0  4
      [11] .plt              PROGBITS        0000000000400480 000480 000040 10  AX  0   0 16
      [12] .text             PROGBITS        00000000004004c0 0004c0 0001c2 00  AX  0   0 16
      [13] .fini             PROGBITS        0000000000400684 000684 000009 00  AX  0   0  4
      [14] .rodata           PROGBITS        0000000000400690 000690 000011 00 AMS  0   0  4
      [15] .eh_frame         PROGBITS        00000000004006a8 0006a8 000114 00   A  0   0  8
      [16] .eh_frame_hdr     PROGBITS        00000000004007bc 0007bc 00003c 00   A  0   0  4
      [17] .jcr              PROGBITS        0000000000401df8 000df8 000008 00  WA  0   0  8
      [18] .fini_array       FINI_ARRAY      0000000000401e00 000e00 000008 00  WA  0   0  8
      [19] .init_array       INIT_ARRAY      0000000000401e08 000e08 000008 00  WA  0   0  8
      [20] .dynamic          DYNAMIC         0000000000401e10 000e10 0001d0 10  WA  4   0  8
      [21] .got              PROGBITS        0000000000401fe0 000fe0 000008 00  WA  0   0  8
      [22] .got.plt          PROGBITS        0000000000401fe8 000fe8 000030 00  WA  0   0  8
      [23] .data             PROGBITS        0000000000402018 001018 000010 00  WA  0   0  8
      [24] .tm_clone_table   PROGBITS        0000000000402028 001028 000000 00  WA  0   0  8
      [25] .bss              NOBITS          0000000000402028 001028 000001 00  WA  0   0  1
      [26] .comment          PROGBITS        0000000000000000 001028 000062 01  MS  0   0  1
      [27] .note.gnu.gold-version NOTE            0000000000000000 00108c 00001c 00      0   0  4
      [28] .rand             PROGBITS        0000000000000000 0010a8 00005f 00      0   0  1
      [29] .shstrtab         STRTAB          0000000000000000 00195b 000122 00      0   0  1
      [30] .symtab           SYMTAB          0000000000000000 001108 000660 18     31  49  8
      [31] .strtab           STRTAB          0000000000000000 001768 0001f3 00      0   0  1

And here is part of disassembly in a .text section.
    
    $ objdump -dj.text -M intel foo | grep -A25 "<main"
    00000000004005c0 <main>:
      4005c0:       55                      push   rbp
      4005c1:       48 89 e5                mov    rbp,rsp
      4005c4:       48 83 ec 10             sub    rsp,0x10
      4005c8:       c7 45 fc 00 00 00 00    mov    DWORD PTR [rbp-0x4],0x0
      4005cf:       e8 0c 00 00 00          call   4005e0 <foo>
      4005d4:       31 c0                   xor    eax,eax
      4005d6:       48 83 c4 10             add    rsp,0x10
      4005da:       5d                      pop    rbp
      4005db:       c3                      ret
      4005dc:       0f 1f 40 00             nop    DWORD PTR [rax+0x0]
    
    00000000004005e0 <foo>:
      4005e0:       55                      push   rbp
      4005e1:       48 89 e5                mov    rbp,rsp
      4005e4:       48 83 ec 10             sub    rsp,0x10
      4005e8:       48 bf 94 06 40 00 00    movabs rdi,0x400694
                    00 00 00
      4005f2:       b0 00                   mov    al,0x0
      4005f4:       e8 b7 fe ff ff          call   4004b0 <printf@plt>
      4005f9:       89 45 fc                mov    DWORD PTR [rbp-0x4],eax
      4005fc:       48 83 c4 10             add    rsp,0x10
      400600:       5d                      pop    rbp
      400601:       c3                      ret
      400602:       66 66 66 66 66 2e 0f    nop WORD PTR cs:[rax+rax*1+0x0]
                    1f 84 00 00 00 00 00

Having a careful look, we could figure out the content of the **MCDataFragment** at line 13 in the mc-dump output contains multiple instructions (28 bytes in size) in the .text section (from 0x4005c0 to 0x4005db) of the _foo_ executable. As you see, the first **MCSection** definitely represents the .text section.  The next fragment is **MCAlignFragment**, which is a 4-byte NOP instruction, followed by another MCDataFragment**_ _**- that is a 34-byte foo() function (from 0x4005e0 to 0x400601). However, interestingly a 14-byte NOP instruction (or alignment) in the foo(), the LLVM backend does not have a corresponding MCAlignFragment. Where does it come from? A good guess is it might be generated by linker because it is the end of our object file, foo.o. Hence we need to explore what those fragments are to get the exact bytes to be emitted by the LLVM backend.

Another point is that each MCDataFragment has (one or more) **MCFixup**. A _**fixup**_ represents a placeholder that requires to be recalculated as further compilation process moves on. That is why all placeholders are initially marked as 0s. The fixup could be resolved either at link time or at load time. The latter is often referred to as a _**relocation**_. In order to avoid the confusion of the term, we explicitly refer to a relocation in an object file as a _**link-time relocation**_ (which the linker handles), and a relocation in an executable as a _**load-time relocation**_ (which the dynamic linker or loader handles). As an example, the yellow-colored lines above show one link-time relocation (line 18; the value ends up with 0x400694) and one load-time relocation (line 21; the final value is 0xfffffeb7). To summarize, _load-time relocations are a subset of link-time relocations, which are a subset of entire fixups_. It is obvious that we need the whole fixups for further fine-grained randomization, which cannot be obtained from relocations or even debugging information because the a resolved fixup is no longer available. Of course most fixups could be reconstructed with binary analysis and disassembly, however we want to avoid them possibly because of incompleteness and inaccuracy.

One interesting observation is that a fixup even could be resolved by the assembler itself at compilation time. In particular, it finalizes the placeholder value during the _**relaxation** _phase (Eli wrote a nice article about it [here](https://eli.thegreenplace.net/2013/01/03/assembler-relaxation)). The line 7 (0xc as the final value) is a good instance of the fixup that has been resolved by the assembler. In this case, the call instruction refers to the function foo() that is 12 bytes away (0x4005d4 + 0xc = 0x4005e0). Based on these fixup examples, we could deduce that a fixup information has three important attributes to update it during transformation properly: a) the location of the fixup, b) the size of the fixup to be de-referenced, and c) the fixup kind: either absolute or relative value. The fixups information is essential because they should be updated as code moves around.

The second MCSection consists of a single MCDataFragment (13 bytes). It is part of .rodata section (0x666f6f20...) at 0x400690. Again, the preceded value 0x01000200 must be somehow created by the linker.
    
    $ readelf -x .rodata foo
    Hex dump of section '.rodata':
      0x00400690 01000200 666f6f20 63616c6c 6564210a ....foo called!.
      0x004006a0 00                                  .

The -M option offers the linker map about how each section has been linked. By passing that option to the gold, we could see a clear view of memory map.
    
    $ clang3 -v -o foo -Wl,-M foo.c
    clang version 3.9.0 (tags/RELEASE_390/final)
    ... (omitted)
     "/usr/bin/ld" -z relro --hash-style=gnu --eh-frame-hdr -m elf_x86_64 -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o foo /usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/5.4.0/crtbegin.o -L/usr/lib/gcc/x86_64-linux-gnu/5.4.0 -L/usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu -L/lib/x86_64-linux-gnu -L/lib/../lib64 -L/usr/lib/x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../.. -L/home/kevinkoo001/Repos/antibloater/llvm-3.9.0/build/bin/../lib -L/lib -L/usr/lib -M /tmp/foo-e3c105.o -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-linux-gnu/5.4.0/crtend.o /usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu/crtn.o
    Archive member included because of file (symbol)
    
    /usr/lib/x86_64-linux-gnu/libc_nonshared.a(elf-init.oS)
                                  /usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu/crt1.o (__libc_csu_init)
    ... (omitted)
    Memory map
    ... (omitted)
    .text           0x00000000004004c0      0x1c2
     .text.unlikely
                    0x00000000004004c0        0x0 /usr/lib/gcc/x86_64-linux-gnu/5.4.0/crtbegin.o
     .text.unlikely
                    0x00000000004004c0        0x0 /usr/lib/x86_64-linux-gnu/libc_nonshared.a(elf-init.oS)
     .text          0x00000000004004c0       0x2a /usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu/crt1.o
                    0x00000000004004c0                _start
     .text          0x00000000004004ea        0x0 /usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu/crti.o
     ** fill        0x00000000004004ea        0x6
     .text          0x00000000004004f0       0xc6 /usr/lib/gcc/x86_64-linux-gnu/5.4.0/crtbegin.o
     ** fill        0x00000000004005b6        0xa
     .text          0x00000000004005c0       0x42 /tmp/foo-e3c105.o
                    0x00000000004005e0                foo
                    0x00000000004005c0                main
     ** fill        0x0000000000400602        0xe
     .text          0x0000000000400610       0x72 /usr/lib/x86_64-linux-gnu/libc_nonshared.a(elf-init.oS)
                    0x0000000000400610                __libc_csu_init
                    0x0000000000400680                __libc_csu_fini
     .text          0x0000000000400682        0x0 /usr/lib/gcc/x86_64-linux-gnu/5.4.0/crtend.o
     .text          0x0000000000400682        0x0 /usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../x86_64-linux-gnu/crtn.o
    ... (omitted)
    .rodata         0x0000000000400690       0x11
     ** merge constants
                    0x0000000000400690        0x4
     ** merge strings
                    0x0000000000400694        0xd
    ... (omitted)

The .text section contains user-defined executable code as well as CRT (C Runtime) code. The line 23-25 explains how foo.c code layout has been formed.  Surprisingly, the line 26 (** fill) shows a 14-byte-long alignment that has not been emitted by the LLVM assembler. Likewise, the .rodata section has 4-byte constants (generated by gold), followed by the strings we saw (i.e., foo called!). For more information on how linker works, David wrote a good article [here](http://www.lurklurk.org/linkers/linkers.html). 

Now let's take another example that contains function pointers.
    
    // gate.c example
    #include <stdio.h>
    
    #define SUCCESS "Key success!"
    #define FAIL    "Key failed!"
    #define EASTEGG "Eastegg!"
    #define GUEST   "Accepted as a guest!"
    
    void success(int n) { printf("%s %d\n", SUCCESS, n); }
    void fail(int n) { printf("%s %d\n", FAIL, n); }
    void eastegg(int n) { printf("%s %d\n", EASTEGG, n); }
    void guest(int n) { printf("%s %d\n", GUEST, n); }
    void (*gate[4])(int) = {success, fail, eastegg, guest};
    
    int main(int argc, char** argv) {
      if (argc < 2 || argc > 2) {
        printf("Usage: %s number to call a function\n", argv[0]);
        return -1;
      }
    
      int num = atoi(argv[1]);
    
      if (num >= 0 && num < 4) {
        (*gate[num])(num);
      }
    
      else {
        printf("You should provide number between 0 and 4\n");
        return -1;
      }
    
      return 0;
    }

The line 13 has four function pointers that call different functions depending on the variable num. As seen, we could examine the emitted code in the same fashion, but this time focus on how to call one of the function pointers (stored in the variable gate) determined at runtime. Let's debug the program with a gdb debugger and a [peda plugin](https://github.com/longld/peda). 
    
    gdb-peda$ b gate.c:23
    Breakpoint 1 at 0x400777: file gate.c, line 23.
    gdb-peda$ r 3
    ...(omitted)
    [-------------------------------------code-----------------------------------]
       0x400767 <main+103>: jl     0x40078d <main+141>
       0x40076d <main+109>: cmp    DWORD PTR [rbp-0x14],0x4
       0x400771 <main+113>: jge    0x40078d <main+141>
    => 0x400777 <main+119>: movsxd rax,DWORD PTR [rbp-0x14]
       0x40077b <main+123>: mov    rax,QWORD PTR [rax*8+0x402030]
       0x400783 <main+131>: mov    edi,DWORD PTR [rbp-0x14]
       0x400786 <main+134>: call   rax
       0x400788 <main+136>: jmp    0x4007ad <main+173>
    [------------------------------------stack-----------------------------------]
    0000| 0x7fffffffe120 --> 0x4007c0 (<__libc_csu_init>:   push   r15)
    0008| 0x7fffffffe128 --> 0x300400500
    0016| 0x7fffffffe130 --> 0x7fffffffe228 --> 0x7fffffffe48b ("~/Tmp/blog/gate")
    0024| 0x7fffffffe138 --> 0x2
    0032| 0x7fffffffe140 --> 0x4007c0 (<__libc_csu_init>:   push   r15)
    0040| 0x7fffffffe148 --> 0x7ffff7a2d830 (<__libc_start_main+240>:       mov    edi,eax)
    0048| 0x7fffffffe150 --> 0x0
    0056| 0x7fffffffe158 --> 0x7fffffffe228 --> 0x7fffffffe48b ("~/Tmp/blog/gate")
    [----------------------------------------------------------------------------]
    Legend: code, data, rodata, value
    
    Breakpoint 1, main (argc=0x2, argv=0x7fffffffe228) at gate.c:23
    23          (*gate[num])(num);

Having a breakpoint at line 23 in the source, the yellow line 9-12 corresponds to call a function pointer (call rax) after setting up the register rax. The rax register holds the input value as a command line argument, and dereference the corresponding value in the function pointer table (located in 0x402030) at 0x40077b. Let's check out what values are stored in that location.
    
    $ readelf -x .data gate
    Hex dump of section '.data':
      0x00402020 00000000 00000000 00000000 00000000 ................
      0x00402030 00064000 00000000 40064000 00000000 ..@.....@.@.....
      0x00402040 80064000 00000000 c0064000 00000000 ..@.......@.....
    
    $ readelf --syms gate | grep FUNC
    ...(omitted)
        74: 0000000000400680    53 FUNC    GLOBAL DEFAULT   12 eastegg
        75: 0000000000400640    53 FUNC    GLOBAL DEFAULT   12 fail
        77: 00000000004006c0    53 FUNC    GLOBAL DEFAULT   12 guest
        78: 0000000000400600    53 FUNC    GLOBAL DEFAULT   12 success

As expected, four values reside in a .data section where each value points to four functions (success()@0x400600, fail@0x400640, eastegg@0x400680,  and guest@0x4006c0) to be referred. All these values are part of fixups, and now they could be within .data section as well as .text.

In the same vein, the fixups could stay in a .rodata section as well. The next sample code contains a switch/case statement that generates a jump table as follow.
    
    // switchcase.c example
    #include <stdio.h>
    
    void f0() { printf("f0 called\n"); }
    void f1() { printf("f1 called\n"); }
    void f2() { printf("f2 called\n"); }
    void f3() { printf("f3 called\n"); }
    void f4() { printf("f4 called\n"); }
    void f5() { printf("f5 called\n"); }
    void f6() { printf("f6 called\n"); }
    void f7() { printf("f7 called\n"); }
    void f8() { printf("f8 called\n"); }
    int def_case() { return -1; }
    
    void switchcase(int select) {
        switch (select) {
            case 0: f0(); break;
            case 1: f1(); break;
            case 2: f2(); break;
            case 3: f3(); break;
            case 4: f4(); break;
            case 5: f5(); break;
            case 6: f6(); break;
            case 7: f7(); break;
            case 8: f8(); break;
            default: def_case(); break;
        }
    }
    
    int main(int argc, char** argv) {
      int s = atoi(argv[1]);
      switchcase(s%9);
      return 0;
    }

The jump table (at 0x400938) in the .rodata section below has 9 elements. Similarly, the register rcx stores the de-referenced value depending on the local variable select from the table (before jmp rcx) where each table entry is the 8-byte value in size. Again, these values (fixups) should be updated for transformation.
    
    $ objdump -dj.text -M intel switchcase | grep -A20 "<switchcase>"
    ...(omitted)
      4007df:       0f 87 68 00 00 00       ja     40084d <switchcase+0x8d>
      4007e5:       48 8b 45 f0             mov    rax,QWORD PTR [rbp-0x10]
      4007e9:       48 8b 0c c5 38 09 40 00 mov    rcx,QWORD PTR [rax*8+0x400938]
      4007f1:       ff e1                   jmp    rcx
      4007f3:       e8 08 fe ff ff          call   400600 <f0>
      4007f8:       e9 58 00 00 00          jmp    400855 <switchcase+0x95>
    ...(omitted)
    
    $ readelf -x .rodata switchcase
    Hex dump of section '.rodata':
      0x00400930 01000200 00000000 f3074000 00000000 ..........@.....
      0x00400940 fd074000 00000000 07084000 00000000 ..@.......@.....
      0x00400950 11084000 00000000 1b084000 00000000 ..@.......@.....
      0x00400960 25084000 00000000 2f084000 00000000 %.@...../.@.....
      0x00400970 39084000 00000000 43084000 00000000 9.@.....C.@.....
      0x00400980 66302063 616c6c65 640a0066 31206361 f0 called..f1 ca
      0x00400990 6c6c6564 0a006632 2063616c 6c65640a lled..f2 called.
      0x004009a0 00663320 63616c6c 65640a00 66342063 .f3 called..f4 c
      0x004009b0 616c6c65 640a0066 35206361 6c6c6564 alled..f5 called
      0x004009c0 0a006636 2063616c 6c65640a 00663720 ..f6 called..f7
      0x004009d0 63616c6c 65640a00 66382063 616c6c65 called..f8 calle
      0x004009e0 640a00                              d..

#### **IV. Obtaining metadata from the LLVM Backend**

* * *

Instead of restating how code randomization works in the paper, I'd like to mention some notable changes in the LLVM backend. But the best way to figure out the backend is to read the actual code with [the documentation from the official LLVM site](https://llvm.org/docs/CodeGenerator.html). Yongli has a long note on the LLVM target-independent code generator [here](http://people.cs.pitt.edu/~yongli/notes/llvm3/LLVM3.html).

As shown in the previous examples, the layout information is essential to obtain function and basic block boundaries for fine-grained code randomization. The LLVM backend operates on internal hierarchical structures in a **_machine code (MC)  framework_**, consisting of _machine functions (MF), machine basic blocks (MBB), and machine instructions (MI)_. The framework then creates a new chunk of binary code, called a **_fragment,_** which is the building block of the section (MCSection). The assembler (MCAssembler) finally assembles various fragments (MCDataFragment,  MCRelaxableFragment and MCAlignmentFragment).  
  
[![](http://dandylife.net/blog/wp-content/uploads/2018/05/fragments.png)](http://dandylife.net/blog/archives/743/fragments)

The figure above (Figure 3 in the paper) illustrates the relationship between the fragments and machine basic blocks in a single function as follows.

  * Data fragments may span consecutive basic blocks. 
  * Relaxable fragments has a branch instruction, including a single fixup.
  * Alignment fragments (padding) could be in between either basic blocks or functions.

I have declared all variables with respect to the bookkeeping information for transformation in [include/llvm/MC/MCAsmInfo.h](https://github.com/kevinkoo001/CCR/blob/master/llvm-3.9.0/include/llvm/MC/MCAsmInfo.h) as below because the class instance could be accessed easily in the LLVM backend. As the unit of the assembly process is the fragment to form a section - decoupled from any logical structure (i.e, MFs or MBBs) - there is no notion of functions and basic blocks under MC layer. Hence it is required to internally label MF and MBB per each instruction.
    
    // class MCAsmInfo() in include/llvm/MC/MCAsmInfo.h
    // (a) MachineBasicBlocks (map)
    //    * MFID_MBBID: <size, offset, # of fixups within MBB, alignments, type, sectionName>
    //    - The type field represents when the block is the end of MF or Object where MBB = 0, MF = 1, and Obj = 2
    //    - The sectionOrdinal field is for C++ only; it tells current BBL belongs to which section!
    mutable std::map<std::string, std::tuple<unsigned, unsigned, unsigned, unsigned, unsigned, std::string>> MachineBasicBlocks;
    //    * MFID: fallThrough-ability
    mutable std::map<std::string, bool> canMBBFallThrough;
    //    * MachineFunctionID: size
    mutable std::map<unsigned, unsigned> MachineFunctionSizes;
    //    - The order of the ID in a binary should be maintained layout because it might be non-sequential.
    mutable std::list<std::string> MBBLayoutOrder;
    
    // (b) Fixups (list)
    //    * <offset, size, isRela, parentID, SymbolRefFixupName, isNewSection, secName, numJTEntries, JTEntrySz>
    //    - The last two elements are jump table information for FixupsText only,
    //      which allows for updating the jump table entries (relative values) with pic/pie-enabled.
    mutable std::list<std::tuple<unsigned, unsigned, bool, std::string, std::string, bool, std::string, unsigned, unsigned>> 
            FixupsText, FixupsRodata, FixupsData, FixupsDataRel, FixupsInitArray; 
    //    - FixupsEhframe, FixupsExceptTable; (Not needed any more as a randomizer directly handles them later on)
    //    - Keep track of the latest ID when parent ID is unavailable
    mutable std::string latestParentID;
    
    // (c) Others
    //     The following method helps full-assembly file (*.s) identify functions and basic blocks
    //     that inherently lacks their boundaries because neither MF nor MBB has been constructed.
    mutable bool isAssemFile = false;
    mutable std::string prevOpcode;
    mutable unsigned assemFuncNo = 0xffffffff;
    mutable unsigned assemBBLNo = 0;
    
      // Update emittedBytes from either DataFragment, RelaxableFragment or AlignFragment
    void updateByteCounter(std::string id, unsigned emittedBytes, unsigned numFixups, \
                           bool isAlign, bool isInline) const {
      // std::string id = std::to_string(fnid) + "_" + std::to_string(bbid);
      // Create the tuple for the MBB
      if (MachineBasicBlocks.count(id) == 0) {
        MachineBasicBlocks[id] = std::make_tuple(0, 0, 0, 0, 0, "");
      }
      
      // Otherwise update MBB tuples
      std::get<0>(MachineBasicBlocks[id]) += emittedBytes; // Acutal size in MBB
      std::get<2>(MachineBasicBlocks[id]) += numFixups;    // Number of Fixups in MBB
      if (isAlign)
        std::get<3>(MachineBasicBlocks[id]) += emittedBytes;  // Count NOPs in MBB
    
      // If inlined, add the bytes in the next MBB instead of current one
      if (isInline)
        std::get<0>(MachineBasicBlocks[latestParentID]) -= emittedBytes;
    }

In order to gather the pre-defined set of metadata for randomization, it is needed to understand code generation in the LLVM backend. The following call stacks help how instructions are emitted. (*) sets up _**the parent of each instruction**_ (MFID_MBBID) and _**fall-through-ability**_.  The property of fall-through is significant when performing randomization at a basic block level because relocating the fall-through BBL renders it unreachable (As we do not insert any trampoline (or instruction) by design, it forms a constraint during BBL-level transformation). (**)  collects the number of bytes of the instruction and the jump table corresponding to a certain fixup. Note that the size of the relaxable fragmentation is postponed until the MCAssember completes [instruction relaxation](https://eli.thegreenplace.net/2013/01/03/assembler-relaxation) process. Check out the source files in my [CCR repository](https://github.com/kevinkoo001/CCR).
    
    --------------------------------------------------------------------------------------------
    llvm::MCELFStreamer::EmitInstToData @lib/MC/MCELFStreamer.cpp (**)
    llvm::MCObjectStreamer::EmitInstruction @lib/MC/MCObjectStreamer.cpp
    --------------------------------------------------------------------------------------------
    llvm::X86AsmPrinter::EmitAndCountInstruction@lib/Target/X86/X86MCInstLower.cpp(*)
    llvm::X86AsmPrinter::EmitInstruction @lib/Target/X86/X86MCInstLower.cpp
    --------------------------------------------------------------------------------------------
    llvm::AsmPrinter::EmitFunctionBody @lib/CodeGen/AsmPrinter/AsmPrinter.cpp
    llvm::X86AsmPrinter::runOnMachineFunction @lib/Target/X86/X86AsmPrinter.cpp
    llvm::MachineFunctionPass::runOnFunction @lib/CodeGen/MachineFunctionPass.cpp
    llvm::FPPassManager::runOnFunction @lib/IR/LegacyPassManager.cpp
    llvm::FPPassManager::runOnModule @lib/IR/LegacyPassManager.cpp
        ::MPPassManager::runOnModule @lib/IR/LegacyPassManager.cpp
    llvm::legacy::PassManagerImpl::run @lib/IR/LegacyPassManager.cpp
    llvm::legacy::PassManager::run @lib/IR/LegacyPassManager.cpp
        ::EmitAssemblyHelper::EmitAssembly@tools/clang/lib/CodeGen/BackendUtil.cpp
    clang::EmitBackendOutput @tools/clang/lib/CodeGen/BackendUtil.cpp              
    clang::BackendConsumer::HandleTranslationUnit@tools/clang/lib/CodeGen/CodeGenAction.cpp    
    clang::ParseAST@tools/clang/lib/Parse/ParseAST.cpp
    clang::ASTFrontendAction::ExecuteAction@tools/clang/lib/Frontend/FrontendAction.cpp        
    clang::CodeGenAction::ExecuteAction @tools/clang/lib/CodeGen/CodeGenAction.cpp
    clang::FrontendAction::Execute @tools/clang/lib/Frontend/FrontendAction.cpp
    clang::CompilerInstance::ExecuteAction@tools/clang/lib/Frontend/CompilerInstance.cpp
    --------------------------------------------------------------------------------------------
    clang::ExecuteCompilerInvocation@tools/clang/lib/FrontendTool/ExecuteCompilerInvocation.cpp
    cc1_main @tools/clang/tools/driver/cc1_main.cpp
    ExecuteCC1Tool @tools/clang/tools/driver/driver.cpp                                         
    --------------------------------------------------------------------------------------------
    main @tools/clang/tools/driver/driver.cpp
    --------------------------------------------------------------------------------------------
    
    // X86AsmPrinter::EmitInstruction() in lib/Target/X86/X86MCInstLower.cpp (*)
    ...(omitted)
    MCInst TmpInst;
    MCInstLowering.Lower(MI, TmpInst);
    
    // Koo [Note] While converting MachineInstr into MCInst, it is essential to maintain
    //            its parent MF and MBB because MCStreamer and MCAssembler do not care 
    //            them any more semantically. After this phase, fragment and section govern.
    const MachineBasicBlock *MBB = MI->getParent();
    unsigned MBBID = MBB->getNumber();
    unsigned MFID = MBB->getParent()->getFunctionNumber();
    std::string ID = std::to_string(MFID) + "_" + std::to_string(MBBID);
    TmpInst.setParent(ID);
      
    // Koo [Note] Simple hack: both MF and MAI can be accessible, thus update fallThrough here.
    const MCAsmInfo *MAI = getMCAsmInfo();
    if (MAI->canMBBFallThrough.count(ID) == 0)
       MAI->canMBBFallThrough[ID] = MF->canMBBFallThrough[ID];
    MAI->latestParentID = ID;
    
    // MCELFStreamer::EmitInstToData() in lib/MC/MCELFStreamer.cpp (**)
    // Add the fixups and data.
    for (unsigned i = 0, e = Fixups.size(); i != e; ++i) {
      Fixups[i].setOffset(Fixups[i].getOffset() + DF->getContents().size());
    
      // Koo: Update a jump table corresponding to a fixup
      Fixups[i].setFixupParentID(ID);
      const MCExpr *FixupExpr = Fixups[i].getValue();
      std::string SymName;
      
      if (FixupExpr->getKind() == MCExpr::SymbolRef || FixupExpr->getKind() == MCExpr::Binary) {
        std::string JTPrefix = ".LJTI";
        if (FixupExpr->getKind() == MCExpr::SymbolRef) {
          const MCSymbolRefExpr &SRE1 = cast<MCSymbolRefExpr>(*FixupExpr);
          const MCSymbol &Sym1 = SRE1.getSymbol();
          SymName = Sym1.getName().str();
        }
        
        // This code is added because of pic/pie option
        //    Fixup kind is "MCExpr::Binary" rather than "MCExpr::SymbolRef"
        //    So here we evaluate BE.getLHS() instead
        if (FixupExpr->getKind() == MCExpr::Binary) {
          const MCBinaryExpr &BE = cast<MCBinaryExpr>(*FixupExpr);
          if (isa<MCSymbolRefExpr>(BE.getLHS())) {
            const MCSymbolRefExpr &SRE2 = cast<MCSymbolRefExpr>(*BE.getLHS());
            const MCSymbol &Sym2 = SRE2.getSymbol();
            SymName = Sym2.getName().str();
          }
        }
        
        // Update the symbol reference for JT only for now.
        if (SymName.find(JTPrefix) != std::string::npos) {
          Fixups[i].setIsJumpTableRef(true);
          Fixups[i].setSymbolRefFixupName(SymName.substr(JTPrefix.length(), SymName.length()));
        }
      }
      DF->getFixups().push_back(Fixups[i]);
    }
    
    DF->setHasInstructions(true);
    DF->getContents().append(Code.begin(), Code.end());
    
    // Koo: Here combines the emitted data as MCDataFragment
    //      addMachineBasicBlockTag() keeps track of the IDs that identifies (MFID_MBBID) pair
    //      MCRelaxableFragment will be generating in MCObjectStreamer::EmitInstToFragment()
    unsigned EmittedBytes = Code.size();
    const MCAsmInfo *MAI = Assembler.getContext().getAsmInfo();
    unsigned numFixups = Fixups.size();
    
    // Sometimes there exists the instruction with missing parentID (!!!!)
    // Another corner case: However, we need to update the emitted bytes anyways
    // For example, "cld; rep; stosq\n" emits 0xFC, (0xF3, 0x48), and 0xAB respectively with no parentID
    if (ID.length() == 0)
      ID = MAI->latestParentID;
    
    DF->setLastParentTag(ID);
    DF->addMachineBasicBlockTag(ID);
    MAI->updateByteCounter(ID, EmittedBytes, numFixups, /*isAlign=*/ false, /*isInline=*/ false);
    
    unsigned size, offset, fixups, alignments, type;
    std::string sectionName;
    std::tie(size, offset, fixups, alignments, type, sectionName) = MAI->MachineBasicBlocks[ID];
    
    MAI->latestParentID = ID;
    ...(omitted)

Next, the jump table information could be collected in [lib/CodeGen/BranchFolding.cpp](https://github.com/kevinkoo001/CCR/blob/master/llvm-3.9.0/lib/CodeGen/BranchFolding.cpp). The tricky part was to spot the final jump table because it keeps updated as optimization goes. In the MF, we walk through all jump table entries, thereby obtain the target MBBs.
    
    // RecordMachineJumpTableInfo() in /lib/CodeGen/MachineFunction.cpp
    
    // Koo - As optimization goes, MJTI might keep being updated from the followings
    //        a) MachineFunctionPass::SelectionDAGISel::X86DAGToDAGISel::runOnMachineFunction() 
    //        b) MachineFunctionPass::BranchFolderPass::runOnMachineFunction() 
    void MachineFunction::RecordMachineJumpTableInfo(MachineJumpTableInfo *MJTI) {
      const std::vector<MachineJumpTableEntry> &JT = MJTI->getJumpTables();
      
      if (!JT.empty()) {
        const MachineModuleInfo &MMI = this->getMMI();
        const MCObjectFileInfo* MOFI = MMI.getMCObjectFileInfo();
        
        if (!MOFI) return;
            
        // Walk through all Jump Tables in this Machine Function
        for (unsigned JTI = 0, e = JT.size(); JTI != e; ++JTI) {
          const std::vector<MachineBasicBlock*> &JTBBs = JT[JTI].MBBs;
          unsigned MFID = this->getFunctionNumber();
          
          // Key: <MachineFunctionIdx_JumpTableIdx>
          std::string MJTKey = std::to_string(MFID) + "_" + std::to_string(JTI); 
          std::list<std::string> JTEntries;
          // Walk through all Jump Table Entries (MBBs) to get targets
          for (unsigned ii = 0, ee = JTBBs.size(); ii != ee; ++ii) {
            unsigned MBBID = JTBBs[ii]->getNumber();
            JTEntries.push_back(std::to_string(MFID) + "_" + std::to_string(MBBID));
          }
        
          // Value: <(EntryKind, EntrySize, Entries[MFID_MBBID])>
          unsigned EntryKind = MJTI->getEntryKind();
          unsigned EntrySize = MJTI->getEntrySize(this->getDataLayout());
          MOFI->updateJumpTableTargets(MJTKey, EntryKind, EntrySize, JTEntries);
        }
      }
    }
    
    // BranchFolderPass::runOnMachineFunction() in lib/CodeGen/BranchFolding.cpp
    ...(omitted)
    // Koo - The final JTInfo, which could be overriden
    //       For example lib/Target/X86/X86ISelDAGToDAG.cpp for x86
    MachineJumpTableInfo *MJTI = MF.getJumpTableInfo();
    if (MJTI)
      MF.RecordMachineJumpTableInfo(MJTI);

MCAssembler performs several important tasks prior to emitting the final binary as follows. It allows for ultimate metadata collection, followed by serializing it to our own section (called .rand). The code snippets below show part of these jobs.

  * Finalize the layout of fragments and sections
  * Attempt to resolve fixups, and records a relocation if unresolved
  * Check if a relaxable fragment needs relaxation
    
    // MCAssembler::layout() in lib/MC/MCAssembler.cpp
    ...(omitted)
    // Koo: Collect what we need once layout has been finalized
    const MCAsmInfo *MAI = Layout.getAssembler().getContext().getAsmInfo();
    const MCObjectFileInfo *MOFI = Layout.getAssembler().getContext().getObjectFileInfo();
    bool isNewTextSection = false, isNewRodataSection = false;
    bool isNewDataSection = false, isNewDataRelSection = false, isNewInitSection = false;
    unsigned textSecCtr = 0, rodataSecCtr = 0, dataSecCtr = 0, dataRelSecCtr = 0, initSecCtr = 0;
    unsigned prevLayoutOrder;
    
    // Evaluate and apply the fixups, generating relocation entries as necessary.
    for (MCSection &Sec : *this) {
      // Koo
      MCSectionELF &ELFSec = static_cast<MCSectionELF &>(Sec);
      std::string secName = ELFSec.getSectionName();
      unsigned layoutOrder = ELFSec.getLayoutOrder();
      
      for (MCFragment &Frag : Sec) {
        // Data and relaxable fragments both have fixups.  So only process
        // those here.
        // FIXME: Is there a better way to do this?  MCEncodedFragmentWithFixups
        // being templated makes this tricky.
        
        // Koo - Attach the size of the alignment to the previous fragment.
        //       Here assumes a) no two alignments are consecutive.
        //                    b) data fragment (DF or RF) exists prior to AF. (may be broken in assembly)
        uint64_t fragOffset = Frag.getOffset();
        MCFragment *prevFrag;
        
        if (isa<MCDataFragment>(&Frag) && Frag.hasInstructions())
          prevFrag = static_cast<MCDataFragment*>(&Frag);
      
        if (isa<MCRelaxableFragment>(&Frag) && Frag.hasInstructions())
          prevFrag = static_cast<MCRelaxableFragment*>(&Frag);
      
        // Update alignment size to reflect to the size of MF and MBB
        if (MOFI->getObjectFileType() == llvm::MCObjectFileInfo::IsELF && \
           secName.find(".text") == 0 && (isa<MCAlignFragment>(&Frag)) && fragOffset > 0) {
           // Push this alignment to the previous MBB and the MF that the MBB belongs to
           unsigned alignSize;
           std::string ID;
           if (isa<MCDataFragment>(*prevFrag))
             ID = static_cast<MCDataFragment*>(prevFrag)->getLastParentTag();
           if (isa<MCRelaxableFragment>(*prevFrag))
             ID = static_cast<MCRelaxableFragment*>(prevFrag)->getInst().getParent();
           
           alignSize = computeFragmentSize(Layout, Frag);
           MAI->updateByteCounter(ID, alignSize, 0, /*isAlign=*/ true, /*isInline=*/ false);
        }
    ...(omitted)
    for (const MCFixup &Fixup : Fixups) {
          uint64_t FixedValue;
          bool IsResolved;
          bool IsPCRel; // Koo
          MCValue Target;
          std::tie(Target, FixedValue, IsResolved) =
          std::tie(Target, FixedValue, IsResolved, IsPCRel) = // Koo
              handleFixup(Layout, Frag, Fixup);
          getBackend().applyFixup(*this, Fixup, Target, Contents, FixedValue,
                                  IsResolved);
                                  
          // Koo: Collect fixups here (ELF format only)
          if (MOFI->getObjectFileType() == llvm::MCObjectFileInfo::IsELF) {
              unsigned offset = fragOffset + Fixup.getOffset();
              unsigned derefSize = 1 << getBackend().getFixupKindLog2Size(Fixup.getKind());
              unsigned jtEntryKind = 0, jtEntrySize = 0, numJTEntries = 0;
              std::map<std::string, std::tuple<unsigned, unsigned, std::list<std::string>>> JTs = MOFI->getJumpTableTargets();
              std::string fixupParentID = Fixup.getFixupParentID();
              std::string SymbolRefFixupName = Fixup.getSymbolRefFixupName();
              std::list<std::string> JTEs; // contains all target(MFID_MBBID) in the JT
              
              // The following handles multiple sections in C++
              if (secName.find(".text") == 0) {
                if (textSecCtr == 0) {
                  prevLayoutOrder = layoutOrder;
                  textSecCtr++;
                }
                else {
                  isNewTextSection = (layoutOrder == prevLayoutOrder) ? false:true;
                  if (isNewTextSection) textSecCtr++;
                  prevLayoutOrder = layoutOrder;
                }
                if (Fixup.getIsJumpTableRef()) {
                  std::tie(jtEntryKind, jtEntrySize, JTEs) = JTs[Fixup.getSymbolRefFixupName()];
                  numJTEntries = JTEs.size();
                }
                MAI->FixupsText.push_back(std::make_tuple(offset, derefSize, IsPCRel, \
                     fixupParentID, SymbolRefFixupName, isNewTextSection, secName, numJTEntries, jtEntrySize));
              }
    ...(omitted)
    
    // MCAssembler::relaxInstruction() in lib/MC/MCAssembler.cpp
    // This method plays a role to determine if relaxation is needed
    
      if (!fragmentNeedsRelaxation(&F, Layout))
      // Koo
      // Whether or not the instruction has been relaxed
      // The RelaxableFragment must be counted as the emitted bytes
      const MCAsmInfo *MAI = Layout.getAssembler().getContext().getAsmInfo();
      std::string ID = F.getInst().getParent();
      unsigned relaxedBytes = F.getRelaxedBytes();
      unsigned fixupCtr = F.getFixup();
      
      if (!fragmentNeedsRelaxation(&F, Layout)) {
        // [Case 1] Unrelaxed instruction
        if (ID.length() > 0) {
          unsigned curBytes = F.getInst().getByteCtr();
          if (relaxedBytes < curBytes) {
            // RelaxableFragment always contains relaxedBytes and fixupCtr variable 
            // for the adjustment in case of re-evaluation (simple hack but tricky)
            MAI->updateByteCounter(ID, curBytes - relaxedBytes, 1 - fixupCtr, 
                                  /*isAlign=*/ false, /*isInline=*/ false);
            F.setRelaxedBytes(curBytes);
            F.setFixup(1);
            
            // If this fixup points to Jump Table Symbol, update it.
            F.getFixups()[0].setFixupParentID(ID);
          }
        }
        return false;
    
      }
    ...(omitted)
      F.getContents() = Code;
      F.getFixups() = Fixups;
    
      // Koo [Case 2] Relaxed instruction: 
      // The only relaxations X86 does is from a 1byte pcrel to a 4byte pcrel
      // Note: The relaxable fragment could be re-evaluated multiple times for relaxation
      //       Thus update it only if the relaxable fragment has not been relaxed previously 
      if (relaxedBytes < Code.size() && ID.length() > 0) {
        MAI->updateByteCounter(ID, Code.size() - relaxedBytes, 1 - fixupCtr, \
                               /*isAlign=*/ false, /*isInline=*/ false);
        F.setRelaxedBytes(Code.size());
        F.setFixup(1);
        F.getFixups()[0].setFixupParentID(ID);
      }
      
      return true;

Lastly, the following code is purely for metadata serialization according to protobuf definition (See the section V) in [lib/MC/MCAssembler.cpp](https://github.com/kevinkoo001/CCR/blob/master/llvm-3.9.0/lib/MC/MCAssembler.cpp). 
    
    // Koo: Serialize the metadata with Google's protocol buffer format, calling from
    //      ELFObjectWriter::writeSectionData() in writeObject()@ELFObjectWriter.cpp
    
    void MCAssembler::WriteRandInfo(const MCAsmLayout &Layout) const {
      ShuffleInfo::ReorderInfo reorder_info;
      serializeReorderInfo(&reorder_info, Layout);
      std::string randContents;
      
      if (!reorder_info.SerializeToString(&randContents)) {
        errs() << "[CCR-Error] MCAssembler::WriteRandInfo - Failed to serialize the shuffling information to .rand section! \n";
      }
      
      MCObjectWriter& OW = (*this).getWriter();
      OW.writeBytes(randContents);
      std::string objFileName(Layout.getAssembler().getContext().getMainFileName());
      
      if (objFileName.length() == 0)
        objFileName = getObjTmpName();
    
      DEBUG_WITH_TYPE("ccr-metadata", dbgs() << "Successfully wrote the metadata in a .rand section for " << objFileName << "\n");
      google::protobuf::ShutdownProtobufLibrary();
    }
    
    // Koo: Serialize all information for future reordering, which has been stored in MCAsmInfo
    void serializeReorderInfo(ShuffleInfo::ReorderInfo* ri, const MCAsmLayout &Layout) {
      // Set the binary information for reordering
      ShuffleInfo::ReorderInfo_BinaryInfo* binaryInfo = ri->mutable_bin();
      binaryInfo->set_rand_obj_offset(0x0); // Should be updated at linking time
      binaryInfo->set_main_addr_offset(0x0);    // Should be updated at linking time 
      
      const MCAsmInfo *MAI = Layout.getAssembler().getContext().getAsmInfo();
      updateReorderInfoValues(Layout);
      
      // Set the layout of both Machine Functions and Machine Basic Blocks with protobuf definition
      std::string sectionName;
      unsigned MBBSize, MBBoffset, numFixups, alignSize, MBBtype;
      unsigned objSz = 0, numFuncs = 0, numBBs = 0;
      int MFID, MBBID, prevMFID = 0;
    
      for (auto MBBI = MAI->MBBLayoutOrder.begin(); MBBI != MAI->MBBLayoutOrder.end(); ++MBBI) {
        ShuffleInfo::ReorderInfo_LayoutInfo* layoutInfo = ri->add_layout();
        std::string ID = *MBBI;
        std::tie(MFID, MBBID) = separateID(ID);
        std::tie(MBBSize, MBBoffset, numFixups, alignSize, MBBtype, sectionName) = MAI->MachineBasicBlocks[ID];
        bool MBBFallThrough = MAI->canMBBFallThrough[ID];
     
        layoutInfo->set_bb_size(MBBSize);
        layoutInfo->set_type(MBBtype);
        layoutInfo->set_num_fixups(numFixups);
        layoutInfo->set_bb_fallthrough(MBBFallThrough);
        layoutInfo->set_section_name(sectionName);
        
        if (MFID > prevMFID) {
          numFuncs++;
          numBBs = 0;
        }
        
        objSz += MBBSize;
        numBBs++;
        prevMFID = MFID;
      }
    
      binaryInfo->set_obj_sz(objSz);
      
      // Set the fixup information (.text, .rodata, .data, .data.rel.ro and .init_array)
      ShuffleInfo::ReorderInfo_FixupInfo* fixupInfo = ri->add_fixup();
      setFixups(MAI->FixupsText, fixupInfo, ".text");
      setFixups(MAI->FixupsRodata, fixupInfo, ".rodata");
      setFixups(MAI->FixupsData, fixupInfo, ".data");
      setFixups(MAI->FixupsDataRel, fixupInfo, ".data.rel.ro");
      setFixups(MAI->FixupsInitArray, fixupInfo, ".init_array");
      
      // Show the fixup information for each section
      DEBUG_WITH_TYPE("ccr-metadata", dbgs() << "\n<Fixups Summary>\n");
      dumpFixups(MAI->FixupsText, "text", /*isDebug*/ false);
      dumpFixups(MAI->FixupsRodata, "rodata", false);
      dumpFixups(MAI->FixupsData, "data", false);
      dumpFixups(MAI->FixupsDataRel, "data.rel.ro", false);
      dumpFixups(MAI->FixupsInitArray, "init_array", false);
    }

#### **V. Metadata Definition with Google's Protocol Buffers**

* * *

We employee [_Google's Protocol Buffers (protobuf)_](https://developers.google.com/protocol-buffers/) to serialize the collected metadata systematically because it provide a clean, efficient and portable interface for structured data streams. As our randomizer has been written in Python, the unified data serialization and de-serialization greatly reduces the complexity to transfer metadata from C++.
    
    // This file defines the buffer protocol of the shuffleInfo for reordering.
    // The following command automatically generates both the declaration 
    // and the implementation of shuffleInfo class.
    //    $ protoc --cpp_out=$DST_DIR shuffleInfo.proto       # C++
    //    $ protoc --python_out=$DST_DIR shuffleInfo.proto    # Python
    // The following command generates the shared object.
    //    $ c++ -fPIC -shared shuffleInfo.pb.cc -o shuffleInfo.so `pkg-config --cflags --libs protobuf`
    
    syntax = "proto2";
    package ShuffleInfo;
    
    message ReorderInfo {
    
      // Binary info from ld or ld.gold; reordering range and main offset
      message BinaryInfo {
        optional uint32 rand_obj_offset = 1;  // PLACEHOLDER FOR LINKER
        optional uint32 main_addr_offset = 2; // PLACEHOLDER FOR LINKER
        optional uint32 obj_sz = 3;           // Verification purpose
      }
    
      // Code Layout Info (.text) from LLVM
      // Embeded info ([#func|#fixup]/obj, [#bbk|#fixup]/func, objSz/ea, funcSz/ea)
      message LayoutInfo {
        optional uint32 bb_size = 1;          // UPDATE AT LINKTIME WITH OBJ ALIGNMENTs
                                              // All alignments between fn/bbl are included here
        optional uint32 type = 2;             // Represents the end of [OBJ|FUN|BBL]
        optional uint32 num_fixups = 3;
        optional bool bb_fallthrough = 4;
        optional string section_name = 5;     // section identifier in c++ mutiple sections
      }
    
      // Fixup info in ELF from LLVM
      message FixupInfo {
        message FixupTuple {
          required uint32 offset = 1;         // UPDATE AT LINKTIME WHEN COMBINING SECTIONS
          required uint32 deref_sz = 2;
          required bool   is_rela = 3;
          optional uint32 type = 4;           // c2c, c2d, d2c, d2d = (0-3)
          optional string section_name = 5;   // section identifier in c++ mutiple sections
                                              // fixup has a jump table (.rodata) for pic/pie use
          optional uint32 num_jt_entries = 6; // number of the jump table entries
          optional uint32 jt_entry_sz = 7;    // size of each jump table entry in byte
        }
        repeated FixupTuple text = 1;
        repeated FixupTuple rodata = 2;
        repeated FixupTuple data = 3;
        repeated FixupTuple datarel = 4;
        repeated FixupTuple initarray = 5;
      }
    
      optional BinaryInfo bin = 1;
      repeated LayoutInfo layout = 2;
      repeated FixupInfo fixup = 3;
    }

The protobuf definition of the metadata uses a compact representation by having the minimum amount of information in need. For instance, the LayoutInfo message only keeps the size of basic block layout with the type of the basic block (The BBL type record denotes whether BBL belongs to the end of a function, the end of an object or both), which will later be reconstructed by the randomizer based on it. Note that section names in LayoutInfo and FixupInfo messages won't be remained in the metadata (.rand section) of the final executable. They are only useful to identify multiple sections for C++ applications at link time. 

#### **VI. Consolidating Metadata in the gold Linker**

* * *

In a nutshell, the main task of the linker is to combine multiple object files generated by compiler into a single executable. It could be broken into three parts: a) constructing final layout, b) resolving symbols, and c) updating relocations. The following figure well illustrates how every metadata per each object file could be merged with appropriate updates (adjustment will be made for BBL sizes, fixup offsets and so forth) as the layout is finalized at link time. [![](http://dandylife.net/blog/wp-content/uploads/2018/05/linking.png)](http://dandylife.net/blog/?attachment_id=824)

#### **VII. Randomizer (dubbed prander)**

* * *

CCR supports fine-grained transformation at a both function and basic block level. But we have opted to maintain some constraints imposed by the code layout in order to strike a balance between efficiency (performance) and effectiveness (randomization entropy). The current choice simplifies reordering process and helps in maintaining spatial locality in caching strategy. To this end, we prioritize basic block reordering at intra-function level, and then proceed with function-level reordering.

[![](http://dandylife.net/blog/wp-content/uploads/2018/05/constraint.png)](http://dandylife.net/blog/?attachment_id=825)

The figure above explains the two constraints mainly due to fixup size: a function that contains a short fixup (i.e,. 1-byte) as part of jump instruction used for tail-call optimization and a basic block that contains any distance-limiting fixup. Let's say the left part represents the original layout, whereas the middle and the right ones correspond to function and basic block reordering, respectively. In this example, suppose that: i) control can fall through from BBL #0 to BBL #1; ii) fixup (a) in FUN #1 refers to a location in a different function (FUN #2.); and iii) fixup (b) corresponds to a single-byte reference from BBL #4 to BBL #3. Basic blocks #0 and #1 are always displaced together due to the first constraint, as also is the case for #3 and #4 due to the third constraint.

The following shows main components of the randomizer (referred to as prander) at a glance. The prander parses the augmented ELF binary, reading metadata (a). It constructs an internal tree data structure (binary - object(s)- function(s) - basic block(s); note that fixup may or may not appear) (b), followed by performing transformation considering constraints based on the structure (c). Finally, it then builds an instrumented (sane) binary after patching all required ELF sections (d).
    
    (a) ElfParser               (b) ReorderInfo        
    +----------------------+    +------------------+   ELF Sections to be patched
    | readElfHeader()      |    | BinaryInfo()     |   +---------------+
    | readMetadata()       |    | ObjectInfo()     |   | .text         |
    | metadataSanityCheck()|    | FunctionInfo()   |   | .data         |
    +----------------------+    | BasicBlockInfo() |   | .rodata       |
                                | FixupInfo()      |   | .data.rel.ro  |
                                +------------------+   | .init_array   |
    (c) ReorderEngine           (d) BinaryBuilder      | .rela.dyn     |
    +----------------------+    +------------------+   | .dynsym       |
    | resolveConstraints() |    | checkOrigBin()   |   | .symtab       |
    | updateFixups()       |    | patchSections()  |-->| .eh_frame     |
    | performTransform()   |    | emitInstBinary() |   | .eh_frame_hdr |
    +----------------------+    +------------------+   +---------------+

[![](http://dandylife.net/blog/wp-content/uploads/2018/05/tree_structure.png)](http://dandylife.net/blog/archives/743/tree_structure)  
Putting all together, the next is a sample output of a program compiled with CCR, [putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).
    
    $ python ./CCR/randomizer/prander.py -s -b ./putty
        ___   ___   __         ___                     _
       / __\ / __\ /__\       / _ \_ __ __ _ _ __   __| | ___ _ __
      / /   / /   / \//_____ / /_)/ '__/ _` | '_ \ / _` |/ _ \ '__|
     / /___/ /___/ _  \_____/ ___/| | | (_| | | | | (_| |  __/ |
     \____/\____/\/ \_/     \/    |_|  \__,_|_| |_|\__,_|\___|_/
    
     Compiler-assisted Code Randomization: Practical Randomizer
     (In the 39th IEEE Symposium on Security & Privacy 2018)
    
     [INFO   ] Reading the metadata from the .rand section... (shuffleInfoReader.py:139)
     [INFO   ]       Offset to the object  : 0x100 (shuffleInfoReader.py:140)
     [INFO   ]       Offset to the main()  : 0x20 (shuffleInfoReader.py:141)
     [INFO   ]       Total Emitted Bytes   : 0x863f0 (shuffleInfoReader.py:142)
     [INFO   ]       Number of Objects     : 79 (shuffleInfoReader.py:143)
     [INFO   ]       Number of Functions   : 1288 (shuffleInfoReader.py:144)
     [INFO   ]       Number of Basic Blocks: 20712 (shuffleInfoReader.py:145)
     [INFO   ]       Fixups in .text : 32467  (shuffleInfoReader.py:56)
     [INFO   ]       Fixups in .rodata       : 4583  (shuffleInfoReader.py:56)
     [INFO   ]       Fixups in .data : 118  (shuffleInfoReader.py:56)
     [INFO   ]       Number of Jump Tables : 85 (shuffleInfoReader.py:173)
     [INFO   ] Building up the layout... (prander.py:43)
                     100% [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]
     [INFO   ] Sanity check for ./putty...  (reorderInfo.py:580)
     [WARNING]       [.text] Fails to discover the reference BBL (Fixup#16116) (reorderInfo.py:470)
     [WARNING]       [.text] Fails to discover the reference BBL (Fixup#16117) (reorderInfo.py:470)
     [WARNING]       [.text] Fails to discover the reference BBL (Fixup#16637) (reorderInfo.py:470)
     [WARNING]       [.text] Fails to discover the reference BBL (Fixup#16638) (reorderInfo.py:470)
     [WARNING]       [.text] Fails to discover the reference BBL (Fixup#16639) (reorderInfo.py:470)
     [WARNING]       [.text] Fails to discover the reference BBL (Fixup#16744) (reorderInfo.py:470)
     [WARNING]       [.text] Fails to discover the reference BBL (Fixup#16745) (reorderInfo.py:470)
     [CRITICAL]      Verification for Fixups in .text section has been failed! (reorderInfo.py:594)
     [INFO   ]       All sanity checks have been PASSED!! (reorderInfo.py:605)
     [INFO   ] Performing reordering (@BBL)... (prander.py:48)
     [INFO   ]       # of Function Constraints: 17 (reorderEngine.py:216)
     [INFO   ] Instrumenting the binary... (prander.py:66)
     [INFO   ]       Processing section [.dynsym] (binaryBuilder.py:694)
                     100% [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]
     [INFO   ]       Processing section [.rela.dyn] (binaryBuilder.py:694)
     [INFO   ]       Processing section [.text] (binaryBuilder.py:694)
                     100% [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]
     [INFO   ]       Processing section [.rodata] (binaryBuilder.py:694)
                     100% [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]
     [INFO   ]       Processing section [.eh_frame] (binaryBuilder.py:694)
     [INFO   ]       Processing section [.eh_frame_hdr] (binaryBuilder.py:694)
     [INFO   ]       Processing section [.init_array] (binaryBuilder.py:694)
     [INFO   ]       Processing section [.data] (binaryBuilder.py:694)
                     100% [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]
     [INFO   ]       Processing section [.symtab] (binaryBuilder.py:694)
                     100% [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]
     [INFO   ] Summary of Binary Instrumentation (report.py:81)
     [INFO   ]       Binary Name       : ./putty_shuffled (report.py:82)
     [INFO   ]       Main() Addr       : 0x48b520 -> 0x48edb0 (report.py:85)
     [INFO   ]       Symbol Patches    : 1288 (.dynsym|.symtab) (report.py:92)
     [INFO   ]       InitArray Patches : 0 (.init_array) (report.py:93)
     [INFO   ]       CIE / FDE         : 2 / 1294 (.eh_frame) (report.py:94)
     [INFO   ]       FDE Patches       : 1288 (.eh_frame) (report.py:95)
     [INFO   ]       Pair Patches      : 1288 (.eh_frame_hdr) (report.py:96)
     [INFO   ]       Original MD5      : 105bc873fab73a4cb6bca9e1746188d3 (report.py:98)
     [INFO   ]       Shuffled MD5      : c795effe6de63f3f96e3f1cd2e4b5fbc (report.py:99)
     [INFO   ]       Shuffled Size     : 0x0863f0 (report.py:100)
     [INFO   ]       Metadata size     : 0x01e766 (report.py:101)
     [INFO   ]       Total Size        : 0x27cbe8 (report.py:102)
     [INFO   ]       File Inc Rate     : 4.784% (report.py:103)
     [INFO   ]       Entropy [LB, UB]  : [10^3392.23, 10^3866.66] possible versions (report.py:77)
     [INFO   ] Total elapsed time: 18.154 sec(s) (prander.py:77)
     [INFO   ] Success!! The log has been saved to ./putty.log (prander.py:162)

#### **VIII. Evaluation (see the paper for more detail)**

* * *

**A. Randomization Overhead  
**

With SPEC CPU2006 benchmark suite (20 C/C++ programs), we generated 20 different variants (with -O2 optimization and no PIC option) including 10 function reordering and 10 basic block reordering. The average overhead was **0.28%** with a 1.37 standard deviation.

**B. Size increase**

Based on the benchmark suite, it was a modest increase of **13.3%** on average to store metadata. Note that the final executable for distribution embeds the compressed metadata with gzip, whereas a variant does not.

[![](http://dandylife.net/blog/wp-content/uploads/2018/05/overhead.png)](http://dandylife.net/blog/?attachment_id=829)

**C. Entropy**  
[![](http://dandylife.net/blog/wp-content/uploads/2018/05/entropy.png)](http://dandylife.net/blog/archives/743/entropy)  
where  
**p**: the number of object files in a binary  
**Fij**: the _j_th function in the _i_th object  
**fi**: the number of functions in the object  
**bij**: the number of basic blocks in the function Fij  
**xij**: the number of basic blocks that has a constraint  
**yj**: the number of functions that has a constraint  
**E**: Entropy with the base 10 logarithm

Finally, my presentation in _Security and Privacy 2018_ is also available.   
[youtube https://www.youtube.com/watch?v=xIAeyLJ0hKw]
