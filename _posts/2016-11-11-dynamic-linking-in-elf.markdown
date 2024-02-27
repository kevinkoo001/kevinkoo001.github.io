---
author: kevinkoo001@gmail.com
comments: true
date: 2016-11-11 20:59:58+00:00
layout: post
link: http://dandylife.net/blog/archives/660
slug: dynamic-linking-in-elf
title: Dynamic Linking in ELF
wordpress_id: 660
categories:
- Miscellaneous Stuff
---

ELF is the binary format that allows for being both executable and linkable. It is de-facto standard in Linux.

**A. Linking Overview**

As the size of the program functionality grows, modulization helps programmers to maintain their code with efficiency. During compilation, an object file is generated per module. Afterwards, a linker (i.e., _ld_ or _ld.gold_) takes one or more object files and combines them into a single executable file, library file, or another object file. A linker plays an pivotal role to resolve the locations like re-targeting absolute jumps.

An object file contains a symbol - a primitive datatype to be identified - that references another object file in nature. There are two symbol kinds: local symbols and external symbols. A local symbol resides in the same object file (often for the relocation purpose at linking time). For the latter, if an external symbol is defined inside the object file it can be called from other modules. If undefined, it requires to find the symbol to which references. The referenced symbol can be located in either another object file or a library file, a collection of object files that can be shared by other executables.

A library can be static or dynamic. If an application employs a symbol in a static library (.a extension), the compiler directly merges the object file that contains the symbol with a final executable. When the object file contains another symbol in another object file, it should be resolved and combined at compilation time as well although the object file is not required by the original program. In case of a dynamic library (shared object or .so extension), the final executable does not embed the object file. Instead, it delays the resolution of undefined symbols until runtime and lets a dynamic linker do its job. Hence, a statically linked executable can run itself without a library at runtime whereas a dynamically linked one cannot.

Here we demystify how dynamic linking works with a simple example for the shared object or position independent code (-fPIC option in a gcc and clang) in x86_64 (AKA AMD64).

**B. Sample Program**

Here is a tiny source code that has two modules (_test.c_ and _func.c_) and one header file (_func.h_). 
    
    // test.c
    #include <stdio.h>
    #include "func.h"
    
    void calc(int x, int y) {
            printf("x + y = %d\n", add(x, y));
            printf("x - y = %d\n", sub(x, y));
    }
    
    int main(void) {
            int a = 10;
            int b = 7;
            calc(10, 7);
            return 0;
    }
    
    // func.c
    #include "func.h"
    
    int add(int x, int y) {
            int a = x;
            int b = y;
            return a + b;
    }
    
    
    int sub(int x, int y) {
            int a = x;
            int b = y;
            return a - b;
    }
    
    // func.h 
    extern int add(int, int);
    extern int sub(int, int);
    extern void calc(int, int);
    
    // Compile the files above with clang
    // $ clang -o main func.c calc.c

**C. ELF Header and Tables for Program Header and Section Header**

ELF consists of three parts: ELF Header, Program Header Table and Section Header Table. 

First off, ELF header is quite self-explantionary with the defined struct itself in _Elf64_Ehdr_. See the comments below.
    
    # define ELF_NIDENT      16
     
    typedef struct {
            uint8_t         e_ident[ELF_NIDENT]; // ELF Identification: See [a]
            Elf32_Half      e_type;      // Object File Type: See [b]
            Elf32_Half      e_machine;   // Required Architecture
            Elf32_Word      e_version;   // Object Version
            Elf32_Addr      e_entry;     // Entry Point VA*
            Elf32_Off       e_phoff;     // Program Header Offset
            Elf32_Off       e_shoff;     // Section Header Offset
            Elf32_Word      e_flags;     // Processor-Specific Flags
            Elf32_Half      e_ehsize;    // ELF Header Size in Bytes
            Elf32_Half      e_phentsize; // Size of each Program Header Entry
            Elf32_Half      e_phnum;     // Number of Program Headers
            Elf32_Half      e_shentsize; // Size of each Section Header Entry
            Elf32_Half      e_shnum;     // Number of Section Headers
            Elf32_Half      e_shstrndx;  // Section Header String Table Index 
    }
    Elf64_Ehdr;
    
    // [a] Elf_Ident Enum
    enum Elf_Ident {
            EI_MAG0         = 0, // 0x7F
            EI_MAG1         = 1, // 'E'
            EI_MAG2         = 2, // 'L'
            EI_MAG3         = 3, // 'F'
            EI_CLASS        = 4, // Architecture (32/64)
            EI_DATA         = 5, // Byte Order: ELFDATA2LSB=0x1, ELFDATA2MSB=0x2
            EI_VERSION      = 6, // File Version
            EI_OSABI        = 7, // OS Specific
            EI_ABIVERSION   = 8, // OS Specific
            EI_PAD          = 9  // Padding (Unused)
    };
     
    # define ELFMAG0      0x7F // e_ident[EI_MAG0]
    # define ELFMAG1      'E'  // e_ident[EI_MAG1]
    # define ELFMAG2      'L'  // e_ident[EI_MAG2]
    # define ELFMAG3      'F'  // e_ident[EI_MAG3]
    # define ELFDATA2LSB  (1)  // Little Endian
    # define ELFCLASS64   (1)  // 64-bit Architecture
    
    // [b] Elf_type structure
    enum Elf_Type {
            ET_NONE         = 0, // Unkown Type
            ET_REL          = 1, // Relocatable File
            ET_EXEC         = 2  // Executable File
            ET_DYN          = 3  // Shared Object File
            ET_CORE         = 4  // Core File
            ET_LOPROC       = 0xff00  // Processor-Specific
            ET_HIPROC       = 0xffff  // Processor-Specific 
    };

The program header table (PHT) describes how a loader maps the binary into virtual address space (or VA) when loading, whereas the section header table (SHT) has the entries of each defined section header when linking. Each mapped region in VA by a PHT entry is often called a segment from a loader view. As is a section by a SHT entry from a linker view.

The final executable file _main_ is shown as following (Figure 1). The linker view on the left shows how each section is stored as a file at offline. The loader view on the right shows how each segment is loaded as a process at runtime. For instance, [S1] is the first section whose size is 0x1c with (A)llocatable by a loader. R, X and W denote readable, executable and writable respectively. On a loader view, there are four major chucks of memory: 0x400000-0x401000 (RX), 0x401000-0x402000 (RW), regions for shared objects and the space for stack and heap.
    
    <Figure 1>
                    <Linker View>                               <Loader View>
                    
    FileOffset                                  VirtualAddr  LOAD (SZ=0x1000)
              .-----------------------------. ------------> .------------------.
       0x0000 |  ELF Header                 |      0x400000 |  ELF_HDR (R)     |
              .-----------------------------.               |------------------|
       0x0040 |  Program Header Table       |      0x400040 |  PHDR (R)        |
              |                             |               |                  |
              |                             |               |                  |
              |                             |               |                  |
              |                             |               |                  |
              .-----------------------------.               |------------------|
       0x0200 | [S1] .interp (0x1c) A       |      0x400200 |  INTERP (R)      |
              .-----------------------------.               |------------------|
       0x021c | [S2] .note.ABI-tag (0x20) A |               |  (RX)            |
              .-----------------------------.               |                  |
       0x0240 | [S3] .dynsym (0xa8) A       |               |                  |
              .-----------------------------.               |                  |
       0x02e8 | [S4] .dynstr (0x89) A       |               |                  |
              .-----------------------------.               |                  |
       0x0378 | [S5] .hash (0x30) A         |               |                  |
              .-----------------------------.               |                  |
       0x03a8 | [S6] .gnu.version (0xe) A   |               |                  |
              .-----------------------------.               |                  |
       0x03b8 | [S7] .gnu.version_r (0x20) A|               |                  |
              .-----------------------------.               |                  |
       0x03d8 | [S8] .rela.dyn (0x18) A     |               |                  |
              .-----------------------------.               |                  |
       0x03f0 | [S9] .rela.plt (0x48) AI    |               |                  |
              .-----------------------------. ------------> |------------------|
       0x0438 | [S10] .init (0x1a) AX       |               |                  |
              .-----------------------------.               |                  |
       0x0460 | [S11] .plt (0x40) AX        |               |                  |
              .-----------------------------.               |                  |
       0x04a0 | [S12] .text (0xa8) AX       |               |                  |
              |                             |               |                  |
              .-----------------------------.               |                  |
       0x06f4 | [S13] .fini (0x9) AX        |               |                  |
              .-----------------------------. ------------> |------------------|
       0x0700 | [S14] .rodata (0x1c) AMS    |               |                  |
              .-----------------------------.               |                  |
       0x0720 | [S15] .eh_frame (0xd4) A    |               |                  |
              .-----------------------------.               |                  |
       0x07f4 | [S16] .eh_frame_hdr (0x2c) A|               |                  |
              .-----------------------------. ------------> .------------------.
       0x0820 | [S17] .dynamic (0x1d) WA    |    |
              |                             |    |          LOAD (SZ=0x1000)
              .-----------------------------.    |          .------------------.
       0x09f0 | [S18] .got (0x8) WA         |    | 0x401000 |                  |
              .-----------------------------.    \--------> .------------------.
       0x09f8 | [S19] .got.plt (0x30) WA    |      0x401820 |  (RW)            |
              .-----------------------------.               |                  |
       0x0a28 | [S20] .data (0x10) WA       |               |                  |
              .-----------------------------.               |                  |
       0x0a38 | [S21] .jcr (0x8) WA         |               |                  |
              .-----------------------------.               |                  |
       0x0a40 | [S22] .tm_clone_tbl (0x0)WA |               |                  |
              .-----------------------------.               |                  |
       0x0a40 | [S23] .fini_array (0x8) WA  |               |                  |
              .-----------------------------.               |                  |
       0x0a48 | [S24] .init_array (0x8) WA  |               |                  |
              .-----------------------------.               |                  |
       0x0a50 | [S25] .bss (0x4) WA         |      0x401a50 |                  |
              .-----------------------------. ---> 0x402000 .------------------.
       0x0a50 | [S26] .comment (0x62) MS    | \
              .-----------------------------. |             .------------------.
       0x0ab4 | [S27] .note.gnu.gold (0x1c) | | 7ffff7a0e000|  Shared Objects  |
              .-----------------------------. |             |  libc, ld, ...   |
       0x0ad0 | [S28] .symtab (0x408)       | |             |                  |
              .-----------------------------. |             .------------------.
       0x0ed8 | [S29] .strtab (0x23f)       | | 
              .-----------------------------. | No mapping in a virtual address
       0x1117 | [S30] .shstrtab (0x118)     | |
              .-----------------------------. |             .------------------.
       0x1230 |  Section Header Table       | |             |  Heap            |
              |   Entry [S0] NULL           | |             |                  |
              |   Entry [S1] .interp        | |             |                  |
              |         .....               | | 7ffffffde000|  Stack           |
              |   Entry [S30] .shstrtab     | |             .------------------.
       0x19f0 .-----------------------------./

A friendly '_readelf_' command illustrates what each segment and section look like by reading the structure. Note that there are a lot of sections appended during compilation. 
    
    // Section to Segment mapping:
    Segment Sections...
    00
    01     .interp
    02     .interp .note.ABI-tag .dynsym .dynstr .hash .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame .eh_frame_hdr
    03     .dynamic .got .got.plt .data .jcr .tm_clone_table .fini_array .init_array .bss
    04     .dynamic
    05     .note.ABI-tag
    06     .eh_frame_hdr
    07
    
    // Section Headers:
      [Nr] Name              Type             Address           Offset
           Size              EntSize          Flags  Link  Info  Align
      [ 0]                   NULL             0000000000000000  00000000
           0000000000000000  0000000000000000           0     0     0
      [ 1] .interp           PROGBITS         0000000000400200  00000200
           000000000000001c  0000000000000000   A       0     0     1
      [ 2] .note.ABI-tag     NOTE             000000000040021c  0000021c
           0000000000000020  0000000000000000   A       0     0     4
      [ 3] .dynsym           DYNSYM           0000000000400240  00000240
           00000000000000a8  0000000000000018   A       4     1     8
      [ 4] .dynstr           STRTAB           00000000004002e8  000002e8
           0000000000000089  0000000000000000   A       0     0     1
      [ 5] .hash             HASH             0000000000400378  00000378
           0000000000000030  0000000000000004   A       3     0     8
      [ 6] .gnu.version      VERSYM           00000000004003a8  000003a8
           000000000000000e  0000000000000002   A       3     0     2
      [ 7] .gnu.version_r    VERNEED          00000000004003b8  000003b8
           0000000000000020  0000000000000000   A       4     1     4
      [ 8] .rela.dyn         RELA             00000000004003d8  000003d8
           0000000000000018  0000000000000018   A       3     0     8
      [ 9] .rela.plt         RELA             00000000004003f0  000003f0
           0000000000000048  0000000000000018  AI       3    11     8
      [10] .init             PROGBITS         0000000000400438  00000438
           000000000000001a  0000000000000000  AX       0     0     4
      [11] .plt              PROGBITS         0000000000400460  00000460
           0000000000000040  0000000000000010  AX       0     0     16
      [12] .text             PROGBITS         00000000004004a0  000004a0
           0000000000000252  0000000000000000  AX       0     0     16
      [13] .fini             PROGBITS         00000000004006f4  000006f4
           0000000000000009  0000000000000000  AX       0     0     4
      [14] .rodata           PROGBITS         0000000000400700  00000700
           000000000000001c  0000000000000000 AMS       0     0     4
      [15] .eh_frame         PROGBITS         0000000000400720  00000720
           00000000000000d4  0000000000000000   A       0     0     8
      [16] .eh_frame_hdr     PROGBITS         00000000004007f4  000007f4
           000000000000002c  0000000000000000   A       0     0     4
      [17] .dynamic          DYNAMIC          0000000000401820  00000820
           00000000000001d0  0000000000000010  WA       4     0     8
      [18] .got              PROGBITS         00000000004019f0  000009f0
           0000000000000008  0000000000000000  WA       0     0     8
      [19] .got.plt          PROGBITS         00000000004019f8  000009f8
           0000000000000030  0000000000000000  WA       0     0     8
      [20] .data             PROGBITS         0000000000401a28  00000a28
           0000000000000010  0000000000000000  WA       0     0     8
      [21] .jcr              PROGBITS         0000000000401a38  00000a38
           0000000000000008  0000000000000000  WA       0     0     8
      [22] .tm_clone_table   PROGBITS         0000000000401a40  00000a40
           0000000000000000  0000000000000000  WA       0     0     8
      [23] .fini_array       FINI_ARRAY       0000000000401a40  00000a40
           0000000000000008  0000000000000000  WA       0     0     8
      [24] .init_array       INIT_ARRAY       0000000000401a48  00000a48
           0000000000000008  0000000000000000  WA       0     0     8
      [25] .bss              NOBITS           0000000000401a50  00000a50
           0000000000000004  0000000000000000  WA       0     0     4
      [26] .comment          PROGBITS         0000000000000000  00000a50
           0000000000000062  0000000000000001  MS       0     0     1
      [27] .note.gnu.gold-ve NOTE             0000000000000000  00000ab4
           000000000000001c  0000000000000000           0     0     4
      [28] .symtab           SYMTAB           0000000000000000  00000ad0
           0000000000000408  0000000000000018          29    22     8
      [29] .strtab           STRTAB           0000000000000000  00000ed8
           000000000000023f  0000000000000000           0     0     1
      [30] .shstrtab         STRTAB           0000000000000000  00001117
           0000000000000118  0000000000000000           0     0     1
    
    Key to Flags:
      W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
      I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
      O (extra OS processing required) o (OS specific), p (processor specific)

**D.  Linking process**

Before moving on, here is what the object looks like (in _crt1.o_). The relocation record shows that there are four locations that cannot be resolved while compilation process. That is why the 4-byte address is empty filled with 0x0s. The first relocation entry at offset 12 has the reference of ___libc_csu_fini_, defined in another object. We can see that __start_ function actually calls our _main_ function at offset 0x20, the entry point of the final executable.
    
    0000000000000000 <_start>:
       0:   31 ed                   xor    ebp,ebp
       2:   49 89 d1                mov    r9,rdx
       5:   5e                      pop    rsi
       6:   48 89 e2                mov    rdx,rsp
       9:   48 83 e4 f0             and    rsp,0xfffffffffffffff0
       d:   50                      push   rax
       e:   54                      push   rsp
       f:   49 c7 c0 00 00 00 00    mov    r8,0x0
      16:   48 c7 c1 00 00 00 00    mov    rcx,0x0
      1d:   48 c7 c7 00 00 00 00    mov    rdi,0x0
      24:   e8 00 00 00 00          call   29 <_start+0x29>
      29:   f4                      hlt
    
    RELOCATION RECORDS FOR [.text]:
    OFFSET           TYPE              VALUE
    0000000000000012 R_X86_64_32S      __libc_csu_fini
    0000000000000019 R_X86_64_32S      __libc_csu_init
    0000000000000020 R_X86_64_32S      main
    0000000000000025 R_X86_64_PC32     __libc_start_main-0x0000000000000004

Now, when the given program is compiled by default, _gcc_ or _clang_ driver combines necessary files (i.e., CRT) to allow a loader to handle it properly.  The Figure 2 illustrates them. (*) means the function is defined in the object file, whereas others are declared outside of the file. (i.e., using _extern_ keyword in C) For example, crt1.o defines __start _in the file  but the function ___libc_start_main_ has to be resolved in a linking time (or maybe later).
    
    <Figure 2>
    .-------------------.-----------------------.---------------------------.----------.
    | crt1.o            | cri.o                 | crtbegin.o                | crtn.o   |
    .-------------------|-----------------------|---------------------------.----------.
    | _start*           | __init*               | deregister_tm_clones *    | .init    |
    | __libc_csu_fini   | __fini*               | register_tm_clones *      | .fini    |
    | __libc_csu_init   | __gmon_start          | __do_global_dtors_aux *   |          |
    | __libc_start_main | _GLOBAL_OFFSET_TABLE_ | frame_dummy *             |          |
    |                   |                       | __TMC_END__               |          |
    |                   |                       | _ITM_deregisterTMCloneTab |          |
    |                   |                       | _ITM_registerTMCloneTable |          |
    |                   |                       | _Jv_RegisterClasses       |          |
    .-------------------------------------------.---------------------------.----------.
    | libc.a                         | crtend.o | func.o                    | test.o   |
    .--------------------------------.----------.---------------------------.----------.
    | __libc_csu_fini (elf-init.o) * | NONE     | add *                     | calc *   |
    | __libc_csu_init (elf-init.o) * |          | sub *                     | main *   |
    | printf (printf.o) *            |          |                           | add      |
    |                                |          |                           | printf   |
    |                                |          |                           | sub      |
    .--------------------------------.----------.---------------------------.----------.

Let's see the layout of all functions from each object file. Figure 3 is part of sections from 10 to 13 in Figure 1. Interesting enough, the layout of all functions in a single object file is not inter-mixed.
    
    <Figure 3>
             |          ...           |
             |========================|
       .init | _init_proc             |  __init (cri.o) + .init (crtn.o)
             |========================|
       .plt  | __libc_start_main      | \ The addresses will be resolved 
             | __gmon_start__         | | at runtime by dynamic linker,
             | _printf                | | the .plt entry is fixed up 
             |========================| / when each function is invoked.
       .text | _start                 |   crt1.o
             |------------------------| \
             | deregister_tm_clones   | |
             | register_tm_clones     | | crtbegin.o
             | __do_global_dtors_aux  | |
             | frame_dummy            | |
             |------------------------| /
             | calc                   | \
             | main                   | | test.o
             |------------------------| /
             | add                    | \
             | sub                    | | func.o
             |------------------------| /
             | __libc_csu_init        | \
             | __libc_csu_fini        | | libc.a (included in a binary)
             |========================| /
        .fini| _term_proc             | __fini (cri.o) + .fini (crtn.o)
             |========================|
             |          ...           |

**E. Dynamic Linking**

When dynamic linking is required (modern compiler set it by default), a compiler generates ._dynamic_ section (section index 17 above). Note that executable files and shared object files have a separate procedure linkage table (PLT).

With the sample we have, the dynamic section contains 24 entries as following. Pay attention to the highlighted, which are required by dynamic linker. The section ._plt.got (PLTGOT)_ is the very place that the final fix-ups are stored by dynamic linker. The _.rela.plt_ (JMPREL) and _.rela.dyn_ (RELA) are the relocation section tables that describe relocation entries.
    
    Dynamic section at offset 0x820 contains 24 entries:
      Tag        Type                    Name/Value
     0x0000000000000003 (PLTGOT)        0x4019f8      // address of .plt.got section
     0x0000000000000002 (PLTRELSZ)      72 (bytes)    // size of .plt.got section
     0x0000000000000017 (JMPREL)        0x4003f0      // address of .rela.plt section
     0x0000000000000014 (PLTREL)        RELA          // relocation type (REL / RELA)
     0x0000000000000007 (RELA)          0x4003d8      // address of .rela.dyn section
     0x0000000000000008 (RELASZ)        24 (bytes)    // total size of .rela.dyn section
     0x0000000000000009 (RELAENT)       24 (bytes)    // size of an entry in .rela.dyn section
     0x0000000000000015 (DEBUG)         0x0
     0x0000000000000006 (SYMTAB)        0x400240      // address of .symtab section
     0x000000000000000b (SYMENT)        24 (bytes)    // size of an entry in .symtab section
     0x0000000000000005 (STRTAB)        0x4002e8      // address of .strtab section
     0x000000000000000a (STRSZ)         137 (bytes)   // size of .strtab section
     0x0000000000000004 (HASH)          0x400378      // address of hash section
     0x0000000000000001 (NEEDED)        Shared library: [libc.so.6] 
     0x000000000000000c (INIT)          0x400438      // address of .init section
     0x000000000000000d (FINI)          0x4006f4      // address of .fini section
     0x000000000000001a (FINI_ARRAY)    0x401a40      // address of .fini_array section
     0x000000000000001c (FINI_ARRAYSZ)  8 (bytes)     // size of ._fini_array
     0x0000000000000019 (INIT_ARRAY)    0x401a48      // address of .init_array section
     0x000000000000001b (INIT_ARRAYSZ)  8 (bytes)     // size of ._fini_array
     0x000000006ffffff0 (VERSYM)        0x4003a8      // address of .gnu.version section
     0x000000006ffffffe (VERNEED)       0x4003b8      // address of .gnu.version_r section
     0x000000006fffffff (VERNEEDNUM)    1             // number of needed versions
     0x0000000000000000 (NULL)          0x0           // end of .dynamic section

Here are the symbols in relocation sections. The type "_R_X86_64_JUMP_SLOT_" means these symbols need to be resolved at runtime by dynamic linker. The offset is the location that resolved reference has to be stored.
    
    Relocation section '.rela.dyn' at offset 0x3d8 contains 1 entries:
      Offset          Info           Type           Sym. Value    Sym. Name + Addend
    0000004019f0  000300000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0
    
    Relocation section '.rela.plt' at offset 0x3f0 contains 3 entries:
      Offset          Info           Type           Sym. Value    Sym. Name + Addend
    000000401a10  000200000007 R_X86_64_JUMP_SLO 0000000000000000 __libc_start_main@GLIBC_2.2.5 + 0
    000000401a18  000300000007 R_X86_64_JUMP_SLO 0000000000000000 __gmon_start__ + 0
    000000401a20  000100000007 R_X86_64_JUMP_SLO 0000000000000000 printf@GLIBC_2.2.5 + 0

The Figure 4 (before resolution) and 5 (after resolution) illustrate how dynamic linker resolves the references on the fly. With disassembly, three symbols are called at 0x400448, 0x4004c4 and 0x4005c7. At first, they are supposed to jump to somewhere in PLT. Again, another jump instruction in PLT corresponds to somewhere in a _.got.plt_. The value in _.got.plt_ has the address of next instruction in _.plt_ that has pointed to itself (+6).

For example, the address of _printf@plt_ is 0x400490, and it jumps to 0x400496 after dereferencing (rip+0x158a is 0x401a20, and 0x400496 is stored in there). Then it pushes 0x2 and jumps to _.plt_ table.
    
    <Figure 4>
    
          .text                           .plt <--------------------------------------\
          .-------------------.           .---------------------------.               |
    4004a0|                   |     400460|   pushq [rip+0x159a]      | <_GOT+8>      |
          |                   |           |---------------------------|               |
          |                   |           |   jmpq  [rip+0x159c]      | <_GOT+16>     |
          |-------------------|           |---------------------------|               |
    400448| call <__gmon_     |     400470|   jmpq  [rip+0x159a]      | <__libc_..@ --|--\
          |  start__@plt>     | /-------> |   push  0x0               |  got.plt>     |  |
          |-------------------| |         |   jmp   0x400460          | --------------|  |
          |-------------------| |         |---------------------------|               |  |
    4004c4| call <__libc_     | |   400480|   jmpq  [rip+0x1592]      | <__gmon_..@ --|--|--\
          |  start_main@plt>  | | /-----> |   push  0x1               |  got.plt      |  |  |
          |-------------------| | |       |   jmp   0x400460          |---------------|  |  |
          |-------------------| | |       |---------------------------|               |  |  |
    4005c7| call <printf@plt> | | | 400490|   jmpq  [rip+0x158a]      | <printf@ -----|--------\
          |-------------------| | | /---> |   push  0x2               |  got.plt>     |  |  |  |
          |                   | | | |     |   jmp   0x400460          | -------------/   |  |  |
          |                   | | | |     .---------------------------.                  |  |  |
          |                   | | | |       <Procedure Linkage Table>                    |  |  |
          |                   | | | |                                                    |  |  |
          |                   | | | |     .got                                           |  |  |
          .-------------------. | | |     .---------------------------.                  |  |  |
                                | | |     |            0x0            |4019f0            |  |  |
          .got.plt sec_hdr      | | |     |---------------------------|                  |  |  |
          .-------------------. | | |     |   (.dynamic) 0x0401820    |4019f8            |  |  |
          | PLTGOT            | | | |     .---------------------------.                  |  |  |
          |                   | | | |          <Global Offset Table>                     |  |  |
          |                   | | | |                                                    |  |  |
          |                   | | | |                                                    |  |  |
          |                   | | | |     .got.plt                                       |  |  |
          |                   | | | |     .---------------------------.                  |  |  |
          .-------------------. | | |     |           0x0             |401a00            |  |  |
                                | | |     |           0x0             |401a08            |  |  |
          .rela.plt sec_hdr     \-|-|---- |         0x400476          |401a10 <----------/  |  |
          .-------------------.   \-|---- |         0x400486          |401a18 <-------------/  |
          | JMPREL            |     \---- |         0x400496          |401a20 <----------------/
          |   r_offset        |           .---------------------------.
          |   r_info          | <-- R_X86_64_JUMP_SLO type
          |   r_addend        |           
          .-------------------.      

Here is the snapshot after the resolution of ___libc_start_main_ and _printf_ at _glibc_. The code for ___gmon_start___ is already in the final executable. (thus 0x400486). At this point, all references are successfully resolved by dynamic linker. Note that the reference is resolved only once when it is called for the first time. 

The address of the routine for the resolution is stored at 0x401a08, which is _dl_runtime_resolve_avx_ in this example.
    
    <Figure 5>
          .text                               .plt <--------------------------------------\
          .-------------------.               .---------------------------.               |
    4004a0|                   |         400460|   pushq [rip+0x159a]      | <_GOT+8>      |
          |                   |               |---------------------------|               |
          |                   |               |   jmpq  [rip+0x159c]      | <_GOT+16>     |
          |-------------------|               |---------------------------|               |
    400448| call <__gmon_     |  /----> 400470|   jmpq  [rip+0x159a]      | <__libc_..@ --|--\
          |  start__@plt>     |--|\           |   push  0x0               |  got.plt>     |  |
          |-------------------|  ||           |   jmp   0x400460          | --------------|  |
          |-------------------|  ||           |---------------------------|               |  |
    4004c4| call <__libc_     | -/\---> 400480|   jmpq  [rip+0x1592]      | <__gmon_..@ --|--|--\
          |  start_main@plt>  |               |   push  0x1               |  got.plt      |  |  |
          |-------------------|               |   jmp   0x400460          |---------------|  |  |
          |-------------------|               |---------------------------|               |  |  |
    4005c7| call <printf@plt> | ------> 400490|   jmpq  [rip+0x158a]      | <printf@ -----|--------\
          |-------------------|               |   push  0x2               |  got.plt>     |  |  |  |
          |                   |               |   jmp   0x400460          | -------------/   |  |  |
          |                   |               .---------------------------.                  |  |  |
          |                   |                 <Procedure Linkage Table>                    |  |  |
          |                   |                                                              |  |  |
          |                   |               .got                                           |  |  |
          .-------------------.               .---------------------------.                  |  |  |
                                              |            0x0            |4019f0            |  |  |
          .got.plt sec_hdr                    |---------------------------|                  |  |  |
          .-------------------.               |    (.dynamic) 0x0401820   |4019f8            |  |  |
          | PLTGOT            |               .---------------------------.                  |  |  |
          |                   |                    <Global Offset Table>                     |  |  |
          |                   |                                                              |  |  |
          |                   |           /-- [dl_runtime_resolve_avx] <------------\        |  |  |
          |                   |           |   .got.plt                              |        |  |  |
          |                   |           \-> .---------------------------.         |        |  |  |
          .-------------------.               |    0x00007ffff7ffe168     |401a00   |        |  |  |
                                              |    0x00007ffff7dee6a0     |401a08 --/        |  |  |
          .rela.plt sec_hdr                   |    0x00007ffff7a2e740     |401a10 <----------/  |  |
          .-------------------.               |    0x0000000000400486     |401a18 <-------------/  |
          | JMPREL            |               |    0x00007ffff7a637b0     |401a20 <----------------/
          |   r_offset        |               .---------------------------.
          |   r_info          |     <-- R_X86_64_JUMP_SLO type
          |   r_addend        |               
          .-------------------.                   

For more curious readers, here are the source files from glibc that defines ___dl_runtime_resolve_ and __dl_fixup_ internally. With several breakpoints in debugging, the routine stores the _link_map_ at _%rdi_ register and the _reloc_index_ at _%rsi_ register. This index is the very one pushed in _.plt_ section.
    
    #define __dl_runtime_resolve at sysdeps/i386/dl-trampoline.S in the glibc-2.24
    In sysdeps/x86_64/dl-trampoline.h, line 64
            # Copy args pushed by PLT in register.
            # %rdi: link_map, %rsi: reloc_index
            mov (LOCAL_STORAGE_AREA + 8)(%BASE), %RSI_LP (a)
            mov LOCAL_STORAGE_AREA(%BASE), %RDI_LP (b)
            call _dl_fixup          # Call resolver. (c)
            mov %RAX_LP, %R11_LP    # Save return value (d)
            
    In sysdeps/elf/dl-runtime.c, line 66
        DL_FIXUP_VALUE_TYPE _dl_fixup (
               # ifdef ELF_MACHINE_RUNTIME_FIXUP_ARGS 
               ELF_MACHINE_RUNTIME_FIXUP_ARGS, # endif
               struct link_map *l, ElfW(Word) reloc_arg) 
    
    gdb-peda$ x/30i 0x00007ffff7dee6a0
       0x7ffff7dee6a0 <_dl_runtime_resolve_avx>:    push   rbx
       0x7ffff7dee6a1 <_dl_runtime_resolve_avx+1>:  mov    rbx,rsp
       0x7ffff7dee6a4 <_dl_runtime_resolve_avx+4>:  and    rsp,0xffffffffffffffe0
       0x7ffff7dee6a8 <_dl_runtime_resolve_avx+8>:  sub    rsp,0x180
       0x7ffff7dee6af <_dl_runtime_resolve_avx+15>: mov    QWORD PTR [rsp+0x140],rax
       0x7ffff7dee6b7 <_dl_runtime_resolve_avx+23>: mov    QWORD PTR [rsp+0x148],rcx
       0x7ffff7dee6bf <_dl_runtime_resolve_avx+31>: mov    QWORD PTR [rsp+0x150],rdx
       0x7ffff7dee6c7 <_dl_runtime_resolve_avx+39>: mov    QWORD PTR [rsp+0x158],rsi
       0x7ffff7dee6cf <_dl_runtime_resolve_avx+47>: mov    QWORD PTR [rsp+0x160],rdi
       0x7ffff7dee6d7 <_dl_runtime_resolve_avx+55>: mov    QWORD PTR [rsp+0x168],r8
       0x7ffff7dee6df <_dl_runtime_resolve_avx+63>: mov    QWORD PTR [rsp+0x170],r9
       0x7ffff7dee6e7 <_dl_runtime_resolve_avx+71>: vmovdqa YMMWORD PTR [rsp],ymm0
       0x7ffff7dee6ec <_dl_runtime_resolve_avx+76>: vmovdqa YMMWORD PTR [rsp+0x20],ymm1
       0x7ffff7dee6f2 <_dl_runtime_resolve_avx+82>: vmovdqa YMMWORD PTR [rsp+0x40],ymm2
       0x7ffff7dee6f8 <_dl_runtime_resolve_avx+88>: vmovdqa YMMWORD PTR [rsp+0x60],ymm3
       0x7ffff7dee6fe <_dl_runtime_resolve_avx+94>: vmovdqa YMMWORD PTR [rsp+0x80],ymm4
       0x7ffff7dee707 <_dl_runtime_resolve_avx+103>:        vmovdqa YMMWORD PTR [rsp+0xa0],ymm5
       0x7ffff7dee710 <_dl_runtime_resolve_avx+112>:        vmovdqa YMMWORD PTR [rsp+0xc0],ymm6
       0x7ffff7dee719 <_dl_runtime_resolve_avx+121>:        vmovdqa YMMWORD PTR [rsp+0xe0],ymm7
       0x7ffff7dee722 <_dl_runtime_resolve_avx+130>:        bndmov [rsp+0x100],bnd0
       0x7ffff7dee72b <_dl_runtime_resolve_avx+139>:        bndmov [rsp+0x110],bnd1
       0x7ffff7dee734 <_dl_runtime_resolve_avx+148>:        bndmov [rsp+0x120],bnd2
       0x7ffff7dee73d <_dl_runtime_resolve_avx+157>:        bndmov [rsp+0x130],bnd3
       0x7ffff7dee746 <_dl_runtime_resolve_avx+166>:        mov    rsi,QWORD PTR [rbx+0x10]
       0x7ffff7dee74a <_dl_runtime_resolve_avx+170>:        mov    rdi,QWORD PTR [rbx+0x8]
       0x7ffff7dee74e <_dl_runtime_resolve_avx+174>:        call   0x7ffff7de6820 <_dl_fixup>
       0x7ffff7dee753 <_dl_runtime_resolve_avx+179>:        mov    r11,rax
       0x7ffff7dee756 <_dl_runtime_resolve_avx+182>:        bndmov bnd3,[rsp+0x130]
       0x7ffff7dee75f <_dl_runtime_resolve_avx+191>:        bndmov bnd2,[rsp+0x120]
       0x7ffff7dee768 <_dl_runtime_resolve_avx+200>:        bndmov bnd1,[rsp+0x110]
    
    gdb-peda$ x/30i 0x7ffff7de6820
       0x7ffff7de6820 <_dl_fixup>:  push   rbx
       0x7ffff7de6821 <_dl_fixup+1>:        mov    r10,rdi
       0x7ffff7de6824 <_dl_fixup+4>:        mov    esi,esi
       0x7ffff7de6826 <_dl_fixup+6>:        lea    rdx,[rsi+rsi*2]
       0x7ffff7de682a <_dl_fixup+10>:       sub    rsp,0x10
       0x7ffff7de682e <_dl_fixup+14>:       mov    rax,QWORD PTR [rdi+0x68] // 69 l_info[DT_STRTAB]
       0x7ffff7de6832 <_dl_fixup+18>:       mov    rdi,QWORD PTR [rax+0x8]  // 69 [rdi = strtab] const char *strtab = (const void *) D_PTR (l, l_info[DT_STRTAB]);
       0x7ffff7de6836 <_dl_fixup+22>:       mov    rax,QWORD PTR [r10+0xf8] // 72 (D_PTR (l, l_info[DT_JMPREL])
       0x7ffff7de683d <_dl_fixup+29>:       mov    rax,QWORD PTR [rax+0x8]  // 72 const PLTREL *const reloc = (const void *) (D_PTR (l, l_info[DT_JMPREL]) + reloc_offset)
       0x7ffff7de6841 <_dl_fixup+33>:       lea    r8,[rax+rdx*8] 
       0x7ffff7de6845 <_dl_fixup+37>:       mov    rax,QWORD PTR [r10+0x70] // 68 (const void *) D_PTR (l, l_info[DT_SYMTAB])
       0x7ffff7de6849 <_dl_fixup+41>:       mov    rcx,QWORD PTR [r8+0x8]   // 73 reloc->r_info
       0x7ffff7de684d <_dl_fixup+45>:       mov    rax,QWORD PTR [rax+0x8]  // 73 const &symtab[ELFW(R_SYM) (reloc->r_info)]
       0x7ffff7de6851 <_dl_fixup+49>:       mov    rdx,rcx  // 73 const ElfW(Sym) *sym = &symtab[ELFW(R_SYM) (reloc->r_info)]
       0x7ffff7de6854 <_dl_fixup+52>:       shr    rdx,0x20 // 73
       0x7ffff7de6858 <_dl_fixup+56>:       lea    rsi,[rdx+rdx*2] // 73
       0x7ffff7de685c <_dl_fixup+60>:       lea    rsi,[rax+rsi*8] // 73 [rsi = sym, 0x400270]
       0x7ffff7de6860 <_dl_fixup+64>:       mov    rax,QWORD PTR [r10] // 74 
       0x7ffff7de6863 <_dl_fixup+67>:       mov    QWORD PTR [rsp+0x8],rsi // 73
       0x7ffff7de6868 <_dl_fixup+72>:       mov    rbx,rax // 74
       0x7ffff7de686b <_dl_fixup+75>:       add    rbx,QWORD PTR [r8] // 74 void *const rel_addr = (void *)(l->l_addr + reloc->r_offset);
       0x7ffff7de686e <_dl_fixup+78>:       cmp    ecx,0x7 // 79
       0x7ffff7de6871 <_dl_fixup+81>:       jne    0x7ffff7de69c7 <_dl_fixup+423> // 79
       0x7ffff7de6877 <_dl_fixup+87>:       test   BYTE PTR [rsi+0x5],0x3
       0x7ffff7de687b <_dl_fixup+91>:       jne    0x7ffff7de6919 <_dl_fixup+249>
       0x7ffff7de6881 <_dl_fixup+97>:       mov    rax,QWORD PTR [r10+0x1c8]

[References]

[http://www.skyfree.org/linux/references/ELF_Format.pdf](http://www.skyfree.org/linux/references/ELF_Format.pdf)  
[https://software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf  
](https://software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf)[https://sourceware.org/git/?p=glibc.git;a=blob_plain;f=elf/elf.h  
](https://sourceware.org/git/?p=glibc.git;a=blob_plain;f=elf/elf.h)[https://sourceware.org/git/?p=glibc.git;a=blob_plain;f=elf/dl-reloc.c  
](https://sourceware.org/git/?p=glibc.git;a=blob_plain;f=elf/dl-reloc.c)[http://www.cs.stevens.edu/~jschauma/810/elf.html](http://www.cs.stevens.edu/~jschauma/810/elf.html)
