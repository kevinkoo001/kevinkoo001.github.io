---
author: kevinkoo001@gmail.com
comments: true
date: 2016-07-31 07:58:36+00:00
layout: post
link: http://dandylife.net/blog/archives/634
slug: cyber-grand-challenge-by-darpa
title: Cyber Grand Challenge by DARPA
wordpress_id: 634
categories:
- Attack &amp; Defense, Cyber Warfare
tags:
- cgc
- cyber grand challenge
- darpa
- event
---

**1. Overview**

2014년 여름, 미 국방성 연구부문을 담당하고 있는 DARPA (Defense Advanced Research Projects Agency)에서 매우 흥미로운 competition event를 진행한다. 이름하여 Cyber Grand Challenge - 자동화된 공격방어 시스템 (automated cyber reasoning system) 하에 상호 간의 공격과 방어를 모두 기계가 실시간으로 담당하게 해 경쟁하는 대회다.

많은 CTF Challenge 대회가 있지만, 사람의 개입없이 순수히 기계만을 이용해 취약점을 찾고, 공격과 방어를 하는 경우는 세계적으로 처음이었다. 게다가 상당히 큰 금액의 상금은 보안인의 관심을 끌기에 충분했다. 

대표링크는 [여기 https://cgc.darpa.mil](https://cgc.darpa.mil) 로 각종 문서와 대회결과를 확인할 수 있다. 대회는 3년간 예선(2014년)과 최종후보 결선전(2015년)을 거쳐 2016년 8월 4일 DEF CON에서 Final 결승전을 치룬다. Final은 7개의 팀이 겨룰 예정이며, 우승자는 무려 20만불의 상금을 거머쥘 수 있다.

작년 USENIX technical session에서 Invite Talk으로 DARPA의 Mike가 두번의 대회를 치르면서 있었던 과정과 결과를 발표했다. 발표자료는 [여기](https://www.usenix.org/sites/default/files/conference/protected-files/sec15_slides_walker.pdf)를 참고하기 바란다.   
  


**UPDATED  (As of Aug. 8, 2016) **  
a. [_Mayhem_ declared the final winner of historic Cyber Grand Challenge](http://www.darpa.mil/news-events/2016-08-04).  
b. Mayhem has been developed as a cyber reasoning system since 2012 by Sang Kil Cha et al., see the paper, [Unleashing MAYHEM on Binary Code](https://users.ece.cmu.edu/~dbrumley/pdf/Cha%20et%20al._2012_Unleashing%20Mayhem%20on%20Binary%20Code.pdf), for more details)[  
](http://repo.cybergrandchallenge.com/cfe/)c. [CFE File Archive](http://repo.cybergrandchallenge.com/cfe/) is available now.

**2. DECREE **

무엇보다도 DARPA는 취약점 자체에 초점을 맞출 수 있도록 기존 리눅스 환경을 변형한 DECREE라는 운영제체를 개발했다. DECREE는 수백 개의 system call이 존재하는 리눅스와는 다르게 실행파일이 다음과 같이 단 7개의 system call만을 이용하도록 설계했다. ([Header 파일 참조](https://github.com/CyberGrandChallenge/libcgc/blob/master/libcgc.h))

  * int transmit(int fd, const void *buf, size_t count, size_t *tx_bytes);
  * int receive(int fd, void *buf, size_t count, size_t *rx_bytes);
  * int fdwait(int nfds, fd_set *readfds, fd_set *writefds);
  * int allocate(size_t length, int is_X, void **addr);
  * int deallocate(void *addr, size_t length);
  * int random(void *buf, size_t count, size_t *rnd_bytes);

새로운 운영체제에서 작동할 수 있는 실행파일을 컴파일할 수 있도록 gcc를 수정하고, 실행파일포맷 또한 기존의 ELF를 기반으로 한 cgc 포맷을 새로 정의했다. 이는 기존의 어떤 운영체제에서도 파일의 실행은 물론 대회에서 발견한 취약점과 exploit 모두 무미의함을 의미한다. 그리고 IPC (Inter-Process Communication) 도 무척 제한적이다. 공유 메모리를 지원하지 않으며, 단순한 양방향 통신만을 지원한다.

[CyberGrandChallenge Github](https://github.com/CyberGrandChallenge/cgc-release-documentation/blob/master/walk-throughs/running-the-vm.md)에 있는 walk-through 문서를 따라 VirtualBox와 vagrant를 설치하고 vbox를 추가한 후 다음과 같이 5개의 VM을 차례로 부팅할 수 있다.
    
    ~/Repos/cgc $ vagrant up
    Bringing machine 'cb' up with 'virtualbox' provider...
    Bringing machine 'ids' up with 'virtualbox' provider...
    Bringing machine 'pov' up with 'virtualbox' provider...
    Bringing machine 'crs' up with 'virtualbox' provider...
    Bringing machine 'ti' up with 'virtualbox' provider...
    [cb] Clearing any previously set forwarded ports...
    [cb] Clearing any previously set network interfaces...
    [cb] Preparing network interfaces based on configuration...
    [cb] Forwarding ports...
    [cb] -- 22 => 2222 (adapter 1)
    [cb] Running 'pre-boot' VM customizations...
    [cb] Booting VM...
    [cb] Waiting for machine to boot. This may take a few minutes...
    [cb] Machine booted and ready!
    [cb] Setting hostname...
    [cb] Configuring and enabling network interfaces...
    [cb] Mounting shared folders...
    [cb] -- /vagrant
    [cb] VM already provisioned. Run `vagrant provision` or use `--provision` to force it
    [ids] Clearing any previously set forwarded ports...
    [ids] Fixed port collision for 22 => 2222. Now on port 2200.
    [ids] Clearing any previously set network interfaces...
    [ids] Preparing network interfaces based on configuration...
    [ids] Forwarding ports...
    [ids] -- 22 => 2200 (adapter 1)
    [ids] Running 'pre-boot' VM customizations...
    [ids] Booting VM...
    [ids] Waiting for machine to boot. This may take a few minutes...
    [ids] Machine booted and ready!
    [ids] Setting hostname...
    [ids] Configuring and enabling network interfaces...
    [ids] Mounting shared folders...
    [ids] -- /vagrant
    [ids] VM already provisioned. Run `vagrant provision` or use `--provision` to force it
    [pov] Clearing any previously set forwarded ports...
    [pov] Fixed port collision for 22 => 2200. Now on port 2201.
    [pov] Clearing any previously set network interfaces...
    [pov] Preparing network interfaces based on configuration...
    [pov] Forwarding ports...
    [pov] -- 22 => 2201 (adapter 1)
    [pov] Running 'pre-boot' VM customizations...
    [pov] Booting VM...
    [pov] Waiting for machine to boot. This may take a few minutes...
    [pov] Machine booted and ready!
    [pov] Setting hostname...
    [pov] Configuring and enabling network interfaces...
    [pov] Mounting shared folders...
    [pov] -- /vagrant
    [pov] VM already provisioned. Run `vagrant provision` or use `--provision` to force it
    [crs] Clearing any previously set forwarded ports...
    [crs] Fixed port collision for 22 => 2201. Now on port 2202.
    [crs] Clearing any previously set network interfaces...
    [crs] Preparing network interfaces based on configuration...
    [crs] Forwarding ports...
    [crs] -- 22 => 2202 (adapter 1)
    [crs] Running 'pre-boot' VM customizations...
    [crs] Booting VM...
    [crs] Waiting for machine to boot. This may take a few minutes...
    [crs] Machine booted and ready!
    [crs] Setting hostname...
    [crs] Configuring and enabling network interfaces...
    [crs] Mounting shared folders...
    [crs] -- /vagrant
    [crs] VM already provisioned. Run `vagrant provision` or use `--provision` to force it
    [ti] Clearing any previously set forwarded ports...
    [ti] Fixed port collision for 22 => 2202. Now on port 2203.
    [ti] Clearing any previously set network interfaces...
    [ti] Preparing network interfaces based on configuration...
    [ti] Forwarding ports...
    [ti] -- 22 => 2203 (adapter 1)
    [ti] Running 'pre-boot' VM customizations...
    [ti] Booting VM...
    [ti] Waiting for machine to boot. This may take a few minutes...
    [ti] Machine booted and ready!
    [ti] Setting hostname...
    [ti] Configuring and enabling network interfaces...
    [ti] Mounting shared folders...
    [ti] -- /vagrant
    [ti] VM already provisioned. Run `vagrant provision` or use `--provision` to force it

다음은 default로 설정되어 있는 CRS서버에 ssh로 연결한 모습이다. 게스트 운영체에제서 /vagrant로 이동하면 자동으로 Vagrant 스크립트에 의해 호스트 운영체제의 현재 디렉토리를 공유하고 있음을 알 수 있다.
    
    ~/Repos/cgc $ vagrant ssh
    Linux crs 3.13.11-ckt32-cgc #1 SMP Tue Jul 12 14:51:24 UTC 2016 i686
    
    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Fri Jul 29 05:31:37 2016 from 10.0.2.2
    vagrant@crs:~$ cd /vagrant/
    vagrant@crs:/vagrant$ ls
    Vagrantfile  peda  samples  vm.json

**3. Challenge Binaries and Utilities**

2015년 Grand Challenge에서 DARPA는 대회용으로 131개의 Compile된 Binary만을 공개했다. 하지만, 현재 Github Repository에서 모든 바이너리의 소스코드와 취약점, PoV (Proof of Vulnerability), 그리고 상세한 설명 (Service Description)을 제공하고 있다. 

131개의 바이너리는 단순하게 만든 운영체제에서 실제 서비스와 유사할 만큼의 복잡도를 가진 서비스를 제공할 수 있다. 72개의 CC 파일 (7000 LOC), 1236개의 헤더 (19만 LOC), 1996개의 C 파일 (20만 LOC), 6000여 개의 함수와 590개의 PoV가 이를 대변한다. 

(1) CGC Executable Format (CGCEF)

CGC 실행 포맷은 처음 15바이트의 헤더가 ELF와 다르다. \x7fCGC로 시작함을 알 수 있다. 
    
    #define CI_NIDENT  16
    typedef struct{
        unsigned char  e_ident[EI_NIDENT];
    #define C_IDENT "\x7fCGC\x01\x01\x01\x43\x01\x00\x00\x00\x00\x00\x00"
        /* ELF vs CGC identification
        * ELF          CGC
        *  0x7f        0x7f
        *  'E'        'C'
        *  'L'        'G'
        *  'F'        'C'
        *  class      1      : '1' translates to 32bit ELF
        *  data        1      : '1' translates to little endian ELF
        *  version    1      : '1' translates to little endian ELF
        *  osabi      \x43    : '1' CGC
        *  abiversion  1      : '1' translates to version 1
        *  padding    random values
        */
        uint16_t    e_type;        /* Must be 2 for executable */
        uint16_t    e_machine;      /* Must be 3 for i386 */
        uint32_t    e_version;      /* Must be 1 */
        uint32_t    e_entry;        /* Virtual address entry point */
        uint32_t    e_phoff;        /* Program Header offset */
        uint32_t    e_shoff;        /* Section Header offset */
        uint32_t    e_flags;        /* Must be 0 */
        uint16_t    e_ehsize;      /* CGC header's size */
        uint16_t    e_phentsize;    /* Program header entry size */
        uint16_t    e_phnum;        /* # program header entries */
        uint16_t    e_shentsize;    /* Section header entry size */
        uint16_t    e_shnum;        /* # section header entries */
        uint16_t    e_shstrndx;    /* sect header # of str table */
    } CGC32_hdr;

CGC 실행포맷은 cgcef-verify라는 도구로 정상여부를 확인할 수 있으며, readelf와 같이 readcgcef로 읽을 수 있다.  PATH 환경변수를 살짝 조정해서 기존의 bintools을 편하게 대체할 수도 있다.
    
    vagrant@crs:/vagrant$ cgcef_verify ./samples/cqe-challenges/CROMU_00001/bin/CROMU_00001
    vagrant@crs:/vagrant$ readcgcef -e ./samples/cqe-challenges/CROMU_00001/bin/CROMU_00001
    CGCEF Header:
      Magic:   7f 43 47 43 01 01 01 43 01 4d 65 72 69 6e 6f 00 
      Class:                             CGCEF32
      Data:                              2's complement, little endian
      Version:                           1 (current)
      OS/ABI:                            CGC
      ABI Version:                       1
      Type:                              EXEC (Executable file)
      Machine:                           Intel i386
      Version:                           0x1
      Entry point address:               0x804a789
      Start of program headers:          52 (bytes into file)
      Start of section headers:          94208 (bytes into file)
      Flags:                             0
      Size of this header:               52 (bytes)
      Size of program headers:           32 (bytes)
      Number of program headers:         3
      Size of section headers:           40 (bytes)
      Number of section headers:         6
      Section header string table index: 5
    
    CGCEf file type is EXEC (Executable file)
    Entry point 0x804a789
    There are 3 program headers, starting at offset 52
    
    Program Headers:
      Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
      PHDR           0x000034 0x08048034 0x08048034 0x00060 0x00060 R   0x4
      LOAD           0x000000 0x08048000 0x08048000 0x02da0 0x02da0 R E 0x1000
      LOAD           0x002da0 0x0804bda0 0x0804bda0 0x14217 0x14217 RW  0x1000
    
     Section to Segment mapping:
      Segment Sections...
       00     
       01     .text .rodata 
       02     .data 
    There are 6 section headers, starting at offset 0x17000:
    
    Section Headers:
      [Nr] Name           Type            Addr     Off    Size   ES Flg Lk Inf Al
      [ 0] (null)         NULL            00000000 000000 000000 00      0   0  0
      [ 1] .text          PROGBITS        080480a0 0000a0 002a03 00  AX  0   0 16
      [ 2] .rodata        PROGBITS        0804aab0 002ab0 0002f0 00   A  0   0 16
      [ 3] .data          PROGBITS        0804bda0 002da0 014217 00  WA  0   0  4
      [ 4] .comment       PROGBITS        00000000 016fb7 00001e 01  MS  0   0  1
      [ 5] .shstrtab      STRTAB          00000000 016fd5 000028 00      0   0  1
    Key to Flags:
      W (write), A (alloc), X (execute), M (merge), S (strings)
      I (info), L (link order), G (group), x (unknown)
      O (extra OS processing required) o (OS specific), p (processor specific)
    
    vagrant@crs:/vagrant$ ls /usr/i386-linux-cgc/bin
    ar            clang-check   lli               llvm-cov        llvm-mc        llvm-rtdyld      nm
    as            clang-format  lli-child-target  llvm-diff       llvm-mcmarkup  llvm-size        objcopy
    bugpoint      clang-tblgen  llvm-ar           llvm-dis        llvm-nm        llvm-stress      objdump
    c-index-test  ld            llvm-as           llvm-dwarfdump  llvm-objdump   llvm-symbolizer  opt
    clang         ld.bfd        llvm-bcanalyzer   llvm-extract    llvm-ranlib    llvm-tblgen      ranlib
    clang++       llc           llvm-config       llvm-link       llvm-readobj   macho-dump       strip
    
    vagrant@crs:/vagrant$ export PATH=/usr/i386-linux-cgc/bin:$PATH
    vagrant@crs:/vagrant$ which objdump
    /usr/i386-linux-cgc/bin/objdump
    
    ./samples/cqe-challenges/CROMU_00001/bin/CROMU_00001: file format cgc32-i386
    architecture: i386, flags 0x00000102:
    EXEC_P, D_PAGED
    start address 0x0804a789

(2) CGCEF Program Headers

CGCEF는 섹션유형이(section type) 4개밖에 없다. 모든 바이너리는 정적으로 연결(Statically Linked)되어 있으며, 동적링킹 (Dynamic Linked)과 thread를 사용하지 않는다. 따라서 TLS (Thread Local Storage) 섹션도 존재하지 않는다.
    
    typedef struct{
        uint32_t        p_type;        /* Section type */
    #define PT_NULL    0              /* Unused header */
    #define PT_LOAD    1              /* Segment loaded into mem */
    #define PT_PHDR    6              /* Program hdr tbl itself */
    #define PT_CGCPOV2  0x6ccccccc      /* CFE Type 2 PoV flag sect */
        uint32_t        p_offset;      /* Offset into the file */
        uint32_t        p_vaddr;        /* Virtual program address */
        uint32_t        p_paddr;        /* Set to zero */
        uint32_t        p_filesz;      /* Section bytes in file */
        uint32_t        p_memsz;        /* Section bytes in memory */
        uint32_t        p_flags;        /* section flags */
    #define PF_X        (1<<0)          /* Mapped executable */
    #define PF_W        (1<<1)          /* Mapped writable */
    #define PF_R        (1<<2)          /* Mapped readable */
        /* Acceptable flag combinations are:
        *        PF_R
        *        PF_R|PF_W
        *        PF_R|PF_X
        *        PF_R|PF_W|PF_X
        */
        uint32_t        p_align;        /* Only used by core dumps */
    } CGC32_Phdr;
    
    vagrant@crs:/vagrant$ objdump -h ./samples/cqe-challenges/CROMU_00001/bin/CROMU_00001
    
    ./samples/cqe-challenges/CROMU_00001/bin/CROMU_00001: file format cgc32-i386
    
    Sections:
    Idx Name          Size      VMA       LMA       File off  Algn
      0 .text         00002a03  080480a0  080480a0  000000a0  2**4
                      CONTENTS, ALLOC, LOAD, READONLY, CODE
      1 .rodata       000002f0  0804aab0  0804aab0  00002ab0  2**4
                      CONTENTS, ALLOC, LOAD, READONLY, DATA
      2 .data         00014217  0804bda0  0804bda0  00002da0  2**2
                      CONTENTS, ALLOC, LOAD, DATA
      3 .comment      0000001e  00000000  00000000  00016fb7  2**0
                      CONTENTS, READONLY

(3) CGCEF Section Headers

섹션헤더는 디버깅할 때 정보 제공용으로만 사용하며, Loader는 이 섹션을 무시하고 로딩한다. 배포용(release)으로 제공하는 바이너리는 보통 이 섹션이 제거되고(stripped) 없다고 보면 된다.
    
    typedef struct {
        uint32_t        sh_name;        /* Name (index into strtab) */
        uint32_t        sh_type;        /* Section type */
    #define SHT_SYMTAB  2              /* Symbol table */
    #define SHT_STRTAB  3              /* String Table */
        uint32_t        sh_flags;
    #define SHT_WRITE  (1<<0)          /* Section is writable */
    #define SHT_ALLOC  (1<<1)          /* Section is in memory */
    #define SHT_EXECINSTR (1<<2)        /* Section contains code */
        uint32_t        sh_addr;        /* Address of section */
        uint32_t        sh_offset;      /* Offset into file */
        uint32_t        sh_size;        /* Section size in file */
        uint32_t        sh_link;
                /* When sh_type is SHT_SYMTAB, sh_link is the index of
                * the associated SHT_STRTAB section
                */
        uint32_t        sh_info;
                /* When sh_type is SHT_SYMTAB, info is one greater
                * than the symbol table index of the last local
                * symbol.
                */
        uint32_t        sh_addralign;  /* Alignment constraints */
        uint32_t        sh_entsize;
                /* Size in bytes of each entry pointed to by this
                * section table */
    } CGC32_Shdr;

(4) IDA Pro modules for CGC

Reversing을 할 때 표준도구처럼 많은 사람들이 애용하는 IDA Pro에서도 Loading할 수 있도 Module을 제공한다. [여기](http://idabook.com/cgc/)서 ([http://idabook.com/cgc/](http://idabook.com/cgc/)) 버전에 맞는 모듈을 다운받아 설치하면 CGC 바이너리를 로딩할 때 자동으로 Loader를 선택해 준다.

(5) How to run CGC binary as a service

일일히 실행하기 번거로우면, 다음과 같이 임의로 서버와 클라이언트를 실행해 확인해 볼 수 있다. 다음과 같이 cqe 바이너리의 poller directory 내에 for-testing과 for-release 두 가지 형태의 많은 sample data로 테스트해 볼 수 있다.
    
    # SEVER SIDE
    vagrant@crs:/vagrant/cqe-challenges/KPRCA_00022/build/release/bin$ cb-server --insecure -p 10000 -d /vagrant/cqe-challenges/CROMU_00015/bin /vagrant/cqe-challenges/CROMU_00015/bin/CROMU_00015
    connection from: 127.0.0.1:4499
    negotiation flag: 0
    getting random seed
    seed: 22733022E8A0D77BB3F089EA9A2DD56593E0373B739737F976F4EFC937BEB1A4FFEAF08F62D708E22C928970F728052F
    stat: /vagrant/cqe-challenges/CROMU_00015/bin/CROMU_00015 filesize 159244
    CB exited (pid: 7672, exit code: 0)
    total children: 1
    total maxrss 96
    total minflt 68
    total utime 0.000000
    total sw-cpu-clock 13506359
    total sw-task-clock 13504409
    CB exited (pid: 7671, exit code: 0)
    
    # CLIENT SIDE
    vagrant@crs:/vagrant/cqe-challenges/CROMU_00015$ cb-replay --host 127.0.0.1 --port 10000 ./poller/for-testing/POLL_00800.xml 
    # CROMU_00015 - ./poller/for-testing/POLL_00800.xml
    # connected to ('127.0.0.1', 10000)
    ok 1 - match: string
    ok 2 - write: sent 2 bytes
    ok 3 - match: string
    ok 4 - write: sent 11 bytes
    ok 5 - match: string
    ok 6 - write: sent 2 bytes
    ok 7 - match: string
    ok 8 - match: string
    ok 9 - write: sent 2 bytes
    ok 10 - match: string
    ok 11 - match: string
    ok 12 - write: sent 2 bytes
    ok 13 - match: string
    ok 14 - write: sent 2 bytes
    ok 15 - match: string
    ok 16 - write: sent 3 bytes
    ok 17 - match: string
    ok 18 - write: sent 5 bytes
    ok 19 - match: string
    ok 20 - write: sent 2 bytes
    ok 21 - match: string
    ok 22 - write: sent 8 bytes
    ok 23 - match: string
    ok 24 - write: sent 2 bytes
    ok 25 - match: string
    ok 26 - write: sent 8 bytes
    ok 27 - match: string
    ok 28 - write: sent 2 bytes
    ok 29 - match: string
    ok 30 - write: sent 9 bytes
    ok 31 - match: string
    ok 32 - write: sent 3 bytes
    ok 33 - match: string
    ok 34 - write: sent 6 bytes
    ok 35 - match: string
    ok 36 - write: sent 2 bytes
    ok 37 - match: string
    ok 38 - write: sent 8 bytes
    ok 39 - match: string
    ok 40 - write: sent 3 bytes
    ok 41 - match: string
    ok 42 - write: sent 9 bytes
    ok 43 - match: string
    ok 44 - write: sent 2 bytes
    ok 45 - match: string
    ok 46 - write: sent 8 bytes
    ok 47 - match: string
    ok 48 - write: sent 2 bytes
    ok 49 - match: string
    ok 50 - write: sent 8 bytes
    ok 51 - match: string
    ok 52 - write: sent 2 bytes
    ok 53 - match: string
    ok 54 - match: string
    ok 55 - write: sent 2 bytes
    ok 56 - match: string
    ok 57 - write: sent 9 bytes
    ok 58 - match: string
    ok 59 - write: sent 2 bytes
    ok 60 - match: string
    ok 61 - write: sent 9 bytes
    ok 62 - match: string
    ok 63 - write: sent 2 bytes
    ok 64 - match: string
    ok 65 - write: sent 8 bytes
    ok 66 - match: string
    ok 67 - write: sent 2 bytes
    ok 68 - match: string
    ok 69 - write: sent 8 bytes
    ok 70 - match: string
    ok 71 - write: sent 3 bytes
    ok 72 - match: string
    ok 73 - write: sent 2 bytes
    ok 74 - match: string
    ok 75 - write: sent 452 bytes
    ok 76 - match: string
    ok 77 - write: sent 2 bytes
    ok 78 - match: string
    ok 79 - match: string
    ok 80 - write: sent 2 bytes
    ok 81 - match: string
    ok 82 - write: sent 2 bytes
    ok 83 - match: string
    ok 84 - write: sent 2 bytes
    ok 85 - match: string
    ok 86 - write: sent 8 bytes
    ok 87 - match: string
    ok 88 - write: sent 2 bytes
    ok 89 - match: string
    ok 90 - write: sent 8 bytes
    ok 91 - match: string
    ok 92 - write: sent 3 bytes
    ok 93 - match: string
    ok 94 - write: sent 8 bytes
    ok 95 - match: string
    ok 96 - write: sent 2 bytes
    ok 97 - match: string
    ok 98 - write: sent 8 bytes
    ok 99 - match: string
    ok 100 - write: sent 2 bytes
    ok 101 - match: string
    ok 102 - write: sent 8 bytes
    ok 103 - match: string
    ok 104 - write: sent 3 bytes
    ok 105 - match: string
    ok 106 - write: sent 5 bytes
    ok 107 - match: string
    ok 108 - write: sent 2 bytes
    ok 109 - match: string
    ok 110 - write: sent 8 bytes
    ok 111 - match: string
    ok 112 - write: sent 2 bytes
    ok 113 - match: string
    ok 114 - write: sent 8 bytes
    ok 115 - match: string
    ok 116 - write: sent 2 bytes
    ok 117 - match: string
    ok 118 - write: sent 8 bytes
    ok 119 - match: string
    ok 120 - write: sent 2 bytes
    ok 121 - match: string
    ok 122 - write: sent 8 bytes
    ok 123 - match: string
    ok 124 - write: sent 2 bytes
    ok 125 - match: string
    ok 126 - write: sent 8 bytes
    ok 127 - match: string
    ok 128 - write: sent 3 bytes
    ok 129 - match: string
    ok 130 - write: sent 12 bytes
    ok 131 - match: string
    ok 132 - write: sent 3 bytes
    ok 133 - match: string
    ok 134 - write: sent 2 bytes
    ok 135 - match: string
    ok 136 - write: sent 13 bytes
    ok 137 - match: string
    ok 138 - write: sent 2 bytes
    ok 139 - match: string
    # tests passed: 139
    # tests failed: 0
    # total tests passed: 139
    # total tests failed: 0
    # polls passed: 1
    # polls failed: 0

또한 pov 디렉토리에는 실제 취약점을 trigger하는 입력값(input)을 가지고 있어 실제 취약한 코드가 어디인지 살펴보기에 안성맞춤이다. 아래 예제는 client와 통신한 후 exit code가 0이 아닌 상태로 종결되었기에 서버 측에서 실제 register status를 보여주고 있다.
    
    # SERVER SIDE
    vagrant@crs:/vagrant/cqe-challenges/KPRCA_00022/build/release/bin$ cb-server --insecure -p 10000 -d /vagrant/cqe-challenges/CROMU_00015/bin /vagrant/cqe-challenges/CROMU_00015/bin/CROMU_00015
    connection from: 127.0.0.1:38142
    negotiation flag: 0
    getting random seed
    seed: 9D7F1CABD61E527C629EEB20FD4A6E75D01A326A24CA2572DB294255EF554755FC498388FCAB7BCDFBAD3AB1B07BB0B6
    stat: /vagrant/cqe-challenges/CROMU_00015/bin/CROMU_00015 filesize 159244
    register states - eax: 0805a000 ecx: 00000000 edx: b7ffe000 ebx: 00000000 esp: baaaad18 ebp: baaaad24 esi: baaaaef0 edi: 00000000 eip: 08058e27
    CB generated signal (pid: 7686, signal: 11)
    total children: 1
    total maxrss 68
    total minflt 20
    total utime 0.000000
    total sw-cpu-clock 1076908
    total sw-task-clock 1208191
    
    # CLIENT SIDE
    vagrant@crs:/vagrant/cqe-challenges/CROMU_00015$ cb-replay --host 127.0.0.1 --port 10000 ./pov/POV_00006.xml  
    # service - ./pov/POV_00006.xml
    # connected to ('127.0.0.1', 10000)
    ok 1 - read length
    ok 2 - write: sent 2 bytes
    ok 3 - read length
    ok 4 - write: sent 29 bytes
    ok 5 - slept 1.000000
    # tests passed: 5
    # tests failed: 0
    # total tests passed: 5
    # total tests failed: 0
    # polls passed: 1
    # polls failed: 0

**4. CWE Distribution for CGC Binaries**

CWE는 [MITRE (https://cwe.mitre.org/)](https://cwe.mitre.org/)에서 제공하는 일종의 취약점 분류(Common Weakness Enumeration)라고 볼 수 있다.  Semi-Final에서 제공한 131개의 바이너리 Description을 기반으로 살펴본 결과 다음 표와 같이 대략 60여개의 분류와 300개 이상의 취약점을 가지고 있음을 알 수 있다.

[[table id=2 /]](http://dandylife.net/blog/archives/634/cgc)

  
TOP 5는 짐작할 수 있듯이 Classic Buffer Overflow, Stack-based Overflow, Heap-based Overflow, Out-of-Bounds Write, Null Pointer Dereferencing이다.

[  
![cgc](http://dandylife.net/blog/wp-content/uploads/2016/07/cgc.png)](http://dandylife.net/blog/archives/634/cgc)
