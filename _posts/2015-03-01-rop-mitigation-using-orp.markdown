---
author: kevinkoo001@gmail.com
comments: false
date: 2015-03-01 07:03:48+00:00
layout: post
link: http://dandylife.net/blog/archives/477
slug: rop-mitigation-using-orp
title: ROP Mitigation using orp
wordpress_id: 477
categories:
- Attack &amp; Defense, Cyber Warfare
tags:
- mitigation
- rop
- system security
---

바로 이전 포스팅에서 ROP 공격을 완화시킬 수 있는 한 방안인 in-place randomization 기법을 소개했는데, 한 번 testing해 봤다. 이 논문의 저자인 Vasilis Pappas는 그 해 rop 관련 방어기법인 kbouncer를 소개해 MS의 Bluehat에서 상금 20만불을 거머쥐기도 했다. ([http://www.microsoft.com/security/bluehatprize](http://www.microsoft.com/security/bluehatprize)/) 여기서 사용하는 [orp](http://nsl.cs.columbia.edu/projects/orp/)라는 도구 역시 Vasilis가 디자인한 상당히 강력한 ROP 방어 기법 중 하나다.

Return-Oriented Programming은 운영체제에서 임의의 데이터(arbitrary data)를 실행할 수 없도록 NX bit, ASLR, DEP 등 다양한 기법을 동원하더라도 우회할 수 있는 방법 중 하나다.  공격자는 이미 실행할 수 있는 영역에 있는 library (windows DLLs)에서 조각나 있는 byte를 모아 gadget으로 구성한 후, eip와 필요한 register를 이용해 공격에 필요한 연산을 수행할 수 있도록 스택을 꾸미는 게 핵심이다.  ROP는 2008년 Blackhat에서도 [Erik Buchanan에 의해 소개](https://www.blackhat.com/presentations/bh-usa-08/Shacham/BH_US_08_Shacham_Return_Oriented_Programming.pdf)되었고 corelan 팀이 2010년 기술한 [exploit writing tutorial](https://www.corelan.be/index.php/2010/06/16/exploit-writing-tutorial-part-10-chaining-dep-with-rop-the-rubikstm-cube/)에서 매우 상세히 다루고 있다.

기존 연구에서 code randomization, 제어 플로우 무결성을 이용하는 방법도 있지만 이는 소스에 디버깅 정보가 있어야 한다. 또한 runtime 시 체크하는 방법론도 제시되었으나 overhead가 커서 실용적이지 못하다. 이 [논문](http://www.cs.columbia.edu/~vpappas/papers/smash.sp12.pdf)의(Smashing the Gadgets: Hindering Return-Oriented Programming Using In-Place Code Randomization) 아이디어는 간단한 편이지만, 방어력은 실용적(practical)이고 효율적(effective)이다. 최종 결과를 보면 전체 gadget으로 이용할 수 있는 코드 중 10% 정도만을 변환해서 80% gadget을 쓸모없게 (useless) 만들었다고 한다.

핵심은 파일크기를 전혀 변경하지 않고 기존 code flow를 전혀 손상하지 않은 채로 같은 코드로 전환(transformation)하는 것이다. 즉 기존코드에서 gadget으로 사용할 수 있는 부분을 다음과 같은 세 가지 형태로 변환하는 과정을 거친다.



	
  * 개별 명령어 치환 (Atomic instruction substitution)
같은 행위를 하는 명령어는 여러 가지 형태로 변형할 수 있음
아래의 경우 같은 연산을 하지만 opcode는 다름
예) add r/m32, r32 ↔ add r32, r/m32
test r/m8, r8 ↔ or r/m8, r8 ↔ or r8, r/m8

	
  * 재배치(reordering): 내부 Basic Block, register 값이 유지되는 레지스터 변경
종속 그래프를 그려서 코드 블록을 재배치함

	
  * Register 재할당 (reassignment)
CFG(Control Flow Graph, 제어 흐름도)를 그려서 상호 간 영향을 미치지 않는 영역(parallel, self-contained regions)을 확인하고 이해 한해서 register를 바꾸는 방법


이 중 첫 번째 단순 치환은 이해하기 쉽다. 다음 코드를 보자.

    
    [ORIGINAL]
    .text:0700106F 55                push    ebp
    .text:07001070 8B EC             mov     ebp, esp
    .text:07001072 83 EC 28          sub     esp, 28h
    
    [PATCHED]
    .text:0700106F 55                push    ebp
    .text:07001070 8B EC             mov     ebp, esp
    .text:07001072 83 C4 D8          add     esp, 0FFFFFFD8h


Highlight 부분인 sub esp, 28h가 add esp, 0FFFFFD8h 로 바뀌었으나 의미는 esp에서 28을 빼거나 -28을 더하라는 것이므로 동일하다.  유사한 방식으로 다음 코드치환도 마찬가지다.

    
    [ORIGINAL]
    .text:0700110A E8 CE FF FF FF        call    sub_70010DD
    .text:0700110F 85 C0                 test    eax, eax
    
    [PATCHED]
    .text:0700110A E8 CE FF FF FF        call    sub_70010DD
    .text:0700110F 09 C0                 or      eax, eax


두 번째, 재배치 부분을 보자. (그림은 논문에서 인용했음)

[![rop1](http://dandylife.net/blog/wp-content/uploads/2015/03/rop1-1024x262.png)](http://dandylife.net/blog/wp-content/uploads/2015/03/rop1.png)

위 그림은 x86 assembly에서 길이가 유동적인(또는 가변적인) instruction/operand 특성 때문에 여러가지로 해석될 수 있음을 나타낸다. 윗부분은 실제 프로그램 코드를 읽어야 하는 순서를 나타내고 있지만, 내부에 있는 코드를 다르게 잘라 읽으면 아래부분의 gadget처럼 전혀 compiler가 의도하지 않는 instruction set이 나올 수 있다. 여기서 위 코드를 보고, 상호 종속적인 부분이 있는지 확인하는 그래프를 그려보면 다음과 같다. 예를 들어 push ebx는 mov ebx, [ecx+0xc]와 종속적이지만 mov eax, [ecx+0x10]과는 독립적이다.

[![rop2](http://dandylife.net/blog/wp-content/uploads/2015/03/rop2.png)](http://dandylife.net/blog/wp-content/uploads/2015/03/rop2.png)

이를 이용해 보면 아래 코드 좌측은 우측과 같이 변형해도 완전히 동일하게 된다. 이 경우 초기 세 번의 push는 마지막 부분 stack에서 pop 연산을 할 때 LIFO (Last-In First-Out) 특성을 고려해 위치만 맞춰주면 된다.

[![rop3](http://dandylife.net/blog/wp-content/uploads/2015/03/rop3.png)](http://dandylife.net/blog/wp-content/uploads/2015/03/rop3.png)



마지막으로 register 재할당 부분은 바로 예제를 보자.

    
    [ORIGINAL]
    .text:0700104E 8B 35 D0 88 01 07       mov     esi, dword_70188D0
    .text:07001054 EB 0A                   jmp     short loc_7001060
    .text:07001056                         ; ---------------------------------
    .text:07001056
    .text:07001056                         loc_7001056:; CODE XREF: sub_7001000+65j 
    .text:07001056 8B 06                   mov     eax, [esi]
    .text:07001058 8B CE                   mov     ecx, esi
    .text:0700105A FF 50 08                call    dword ptr [eax+8]
    .text:0700105D 8B 76 04                mov     esi, [esi+4]
    .text:07001060
    .text:07001060                         loc_7001060:; CODE XREF: sub_7001000+54j
    .text:07001060 8B 45 EC                mov     eax, [ebp+var_14]
    .text:07001063 3B F0                   cmp     esi, eax
    .text:07001065 75 EF                   jnz     short loc_7001056
    .text:07001067 32 C0                   xor     al, al
    .text:07001069
    .text:07001069                         loc_7001069:  CODE XREF: sub_7001000+4Cj
    .text:07001069 E8 55 55 00 00          call    __EH_epilog3
    .text:0700106E C3                      retn
    
    [PATCHED]
    .text:0700104E 8B 1D D0 88 01 07       mov     ebx, dword_70188D0
    .text:07001054 EB 0A                   jmp     short loc_7001060
    .text:07001056                         ; ---------------------------------
    .text:07001056
    .text:07001056                         loc_7001056: ; CODE XREF: sub_7001000+65j
    .text:07001056 8B 03                   mov     eax, [ebx]
    .text:07001058 8B CB                   mov     ecx, ebx
    .text:0700105A FF 50 08                call    dword ptr [eax+8]
    .text:0700105D 8B 5B 04                mov     ebx, [ebx+4]
    .text:07001060
    .text:07001060                         loc_7001060: ; CODE XREF: sub_7001000+54j
    .text:07001060 8B 45 EC                mov     eax, [ebp-14h]
    .text:07001063 3B D8                   cmp     ebx, eax
    .text:07001065 75 EF                   jnz     short loc_7001056
    .text:07001067 32 C0                   xor     al, al
    .text:07001069
    .text:07001069                         loc_7001069: ; CODE XREF: sub_7001000+4Cj
    .text:07001069 E8 55 55 00 00          call    sub_70065C3
    .text:0700106E C3                      retn


이 경우 원래 esi를 전체 흐름에서 사용하고 있지 않던 ebx로 재할당했다. CPU의 입장에서 보면 한 thread에서 사용하고 있지 않은 다른 저장소를 이용한 셈이므로 전혀 문제될 것이 없다. 하지만 ROP 공격 관점에서 보면, gadget이 없어지거나 사용할 수 없게끔 흐트러지는 결과를 낳는다.

orp는 현재 [-d|-p|-r|-c] 네 가지 option을 사용할 수 있으며, 우선 -d 옵션으로 대상을 IDA Pro를 이용해 분석하는 작업을 한다. 그런 후 -r 옵션으로 randomization하면 된다. -p와 -c는 각각 profiling, coverage evaluation을 의미한다. 아래 결과는 Adobe Reader 9.3 버전에 있는 BIB.dll을 대상으로 1,737개의 명령어가 변경되었음을 알려준다.

    
    python orp.py -r BIB.dll
    Orp v0.3
         found 112 level-0 functions
         found 77 level-1 functions
         found 66 level-2 functions
         found 1 level-3 functions
    no more to analyze .. let's search for typed
    found 66 typed and not analyzed functions
         found 66 level-4 functions
         found 14 level-5 functions
         found 1 level-6 functions
    no more to analyze .. let's search for typed
    found 0 typed and not analyzed functions
         classified 337 out of 1298 functions
         analyzing level-0 functions
         analyzing level-1 functions
         analyzing level-2 functions
         analyzing level-3 functions
         analyzing level-4 functions
         analyzing level-5 functions
         analyzing level-6 functions
         analyzing unclassified functions
    counting the number of updated call instructions:
         total 1737, updated 1533
    changed 2019 bytes of at least 18549 changeable
    (not counting all possible reorderings and preservations)


아래는 변경된 바이트를 byte 단위로 diff한 모습이다.

[![rop4](http://dandylife.net/blog/wp-content/uploads/2015/03/rop4.png)](http://dandylife.net/blog/wp-content/uploads/2015/03/rop4.png)




