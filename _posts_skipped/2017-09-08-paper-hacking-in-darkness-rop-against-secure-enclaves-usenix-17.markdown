---
author: kevinkoo001@gmail.com
comments: true
date: 2017-09-08 22:16:29+00:00
layout: post
link: http://dandylife.net/blog/archives/699
slug: paper-hacking-in-darkness-rop-against-secure-enclaves-usenix-17
title: '[Paper] Hacking in Darkness: ROP against Secure Enclaves (USENIX ''17)'
wordpress_id: 699
categories:
- Attack &amp; Defense, Cyber Warfare
- Papers in Security
---

지난 달 캐나다에서 열린 USENIX 2017에서 발표한 여러 논문들 중에 개인적으로 관심있게 본 논문 하나가 있다. 학계에서는 2007년 처음으로 ROP (return oriented programming) 공격방식이 등장했으니 10년이 지난 지금은 고전처럼 보일지 모르지만, 실행과 쓰기 권한을 동시에 가질 수 없는 정책을 기본으로 하는 현대 운영체제에서 아직도 상당히 강력한 공격방식이다. ASLR을 기본적으로 탑재하면서 코드 재사용 공격 자체가 살짝 주춤했지만, 실행 중 포인터 등을 유출하는 기법(information leak)으로 우회하는 방식으로 선회했다.

이 논문은 하드웨어로 무장한 Trusted Computing 기반에서 이 공격을 수행할 수 있음을 보였기에 상당히 흥미로운 주제가 아닐 수 없다. 우선 Intel이 최근 선보인 SGX(Software Guard eXtenstions)라는 하드웨어 기반에서 작동하는 방식이므로 먼저 SGX에 대해 간단히 알아보고 공격방식을 살펴보자.

**A. 배경지식 (Background)**

인텔의 SGX는 x86 명령어 셋 (Instruction Set Architecture, ISA)을 확장해서 신뢰된 실행 환경(trusted execution environments, TEE)을 생성해 주는 기술이다. 이 환경을 특히 enclave라고 부른다. Enclave는 프로세스의 가상 메모리의 일부만을 격리시켜 실행시간(runtime) 동안 Enclave 내의 신뢰된 application만이 실행할 수 있도록 해당 메모리 공간을 외부로부터 보호한다.

특히 CPU 내에 _메모리 암호화 엔진 (Memory Encryption Engine, MEE)_ 부분을 거쳐야만 메모리를 읽고 쓸 수 있으며, Enclave가 사용하는 가상 메모리는 모두 암호화되어 있다. 하드웨어에 존재하는 키로 복호화해서만 값을 읽고 쓸 수 있어 사실상 운영체제 커널이 변조되었거나 공격을 당할 경우나 cold-boot 등의 많은 물리적 공격으로부터 안전하다.

Enclave가 사용하는 가상 메모리 내부에 Entry Table을 비롯해 전용 Code, Stack, Heap 공간이 존재하며, 그 외 메모리는 운영체제가 전적으로 사용하는 비신뢰 구간(un-trusted zone)이다. Enclave 내부의 application 코드와 데이터 모두 암호화된 상태로 로드하기 때문에 외부 소프트웨어로부터 중요한 데이터 (키나 계정 등)를 완전히 보호할 수 있다. 그럼 Trusted App과 Un-trusted App 사이의 상호작용은 어떻게 할 수 있을까?[![](http://dandylife.net/blog/wp-content/uploads/2017/09/overall2.png)](http://dandylife.net/blog/archives/699/overall2)

위 그림에서 Un-trusted 코드가 유일하게 신뢰된 메모리 영역으로 진입할 수 있는 지점은 바로 _EENTER_라는 명령어를 통해서만 가능한데, 해당 명령어는 보호되어 있는 _enclave 페이지 캐시 (Enclave Page Cache, EPC)_에 있는 코드로 제어권을 넘긴다. 이후 enclave 내부의 application으로 stack pointer가 이동하게 되며, enclave 프로그램이 실행된다. 실행이 종료되면 다시 _EEXIT_이라는 명령어를 통해 스택 포인터를 가리킴으로써 un-trusted 영역으로 빠져나올 수 있는 메카니즘이다. 다시 말해 EENTER과 EEXIT은 서로 다른 두 Zone을 넘나들 수 있게 하는 통로인 셈이다. 

Enclave 내에 있는 함수를 호출하려면, rbx 레지스터에 _쓰레드 제어 구조체(Thread Control Structure, TCS)_를 저장하고, 인자로 untrusted 메모리에 있는 주소값을 포인터로 넘겨준다. 

**B. 위협 모델 (Threat Model)**

기본적으로 SGX를 지원하는 하드웨어에 대한 소프트웨어 기반의 공격을 가정하며, 따라서 부채널(side-channel)을 통한 공격 등은 위협 모델에 포함하지 않으며 다음 사항을 따른다.

  * 공격자의 경우
    * 신뢰할 수 없는 application과 운영체제 등 모든 소프트웨어 제어 가능함
    * 프로그램 행위를 관찰하여 Enclave 프로그램을 반복적으로 crash할 수 있음
  * enclave 응용프로그램
    * Intel SDK를 지원하는 표준 컴파일러로 생성됨
    * 페이지 권한 등 모든 설정은 정상적임
    * 메모리 손상 (memory corruption) 취약성을 가지고 있음
    * 암호화된 형식으로 배포되어 모든 enclave 정보는 실행시 알 수 없음

**C. 공격 세부과정 (Attack Scenario)**

우선 대표적인 memory corruption 공격 중 하나인 버퍼 오버플로우 취약성이 존재하는 코드를 발견한다. 이를 위해 함수 진입점 (entry point) 정보를 우선 수집해 함수에서 사용하는 인자를 반복적으로 fuzzing 공격한다. 공격 과정을 설명하기 위한 아래 모든 그림 자료는 카이스트의 이재혁 외 저자 논문 또는 발표자료에서 발췌했음을 밝힌다.

[![](http://dandylife.net/blog/wp-content/uploads/2017/09/step1.png)](http://dandylife.net/blog/archives/699/step1-2)

Enclave 프로그램은 기본적으로 사용자 권한으로 동작한다. 이는 페이지 폴트(page fault)와 같이 프로세서가 처리해야 할 예외사항 (exception)이 발생했을 때 에러 처리(error handling)를 할 수 없음을 의미한다. 이럴 경우 Enclave application은 예외 처리를 위해 외부로 (이 경우 untrusted한 운영체제) AEX (Asynchronous Enclave Exit) 명령어를 통해 제어권(fallback routine)을 돌려준다. 여기서 성공적인 공격을 위해 AEX handler가 이를 처리할 때 CR2 레지스터를 사용한다는 점을 공략한다. CR2 레지스터는 페이지 폴트가 난 주소 (source address)를 저장하고 있다. 

**[Step 1] gadget 후보 찾기**: 위 그림을 보면, Enclave Stack에는 공격자가 임의로 작성한 공격 payload로 인해 버퍼 오버플로우된 상태이다. 우선 ROP 공격에 필요한 gadget을 찾아야 한다. 여기서 찾고자 하는 gadget 유형은 _pop [reg]; ... ret_ 형태다. 공격 payload에는 페이지 폴트난 주소가 CR2와 같은 경우 공격 payload의 PF_Region 값을 이용해 pop [reg]가 몇 개인지 먼저 확인할 수 있다. 예를 들어 (1)에서 후보 gadget 중 하나로 return 하면, _pop _이 하나인 gadget을 만날 수 있다. 현재 스택 포인터 (rsp)는 PF_Region_0을 가리키고 있을 테고, ret을 만나면 (2) PF_Region_1의 값을 rip (또는 PC; Program Counter) 에 넣어 해당주소를 실행하고자 (3) 할 것이다. 이 경우 실행할 수 있는 영역이 아니라면 (4)와 같이 page fault가 난다. 자, 지금까지 알아낸 사실은 gadget 후보를 발견했고, 해당 gadget은 pop이 몇 개나 반복된 후 ret하는지까지다. 지금 단계에서는 어떤 register인지 알 길이 없다.

[![](http://dandylife.net/blog/wp-content/uploads/2017/09/enclu.png)](http://dandylife.net/blog/archives/699/enclu)

다음 단계로 넘어가기 전에 몇 가지 전형적인 조건 (가정사항)이 있다. Enclave 프로그램은 최소 하나 이상의 _ENCLU 명령어 (0x0F 0x01 0xD7)_ 를 가지고 있어야 한다. ENCLU 명령어는 사용자 권한으로 실행되며, 하나의 opcode로 복수 개의 기능을 수행할 수 있는 독특한 명령어다. 이 기능은 rax 값에 따라 정해지며, 각 기능은 노드 함수(leaf function)를 통해 수행한다. 예를 들어 위 그림에서 rax 값이 0x4인 경우 enclave를 exit하는 노드 함수를 호출하고, 0x1인 경우 암호키를 받아오게 된다. 

두 번째로 enclave 내부에 있는 application은 외부 untrusted 프로그램과 상호작용하기 위해 자주 값을 양쪽으로 복사하는 행위를 한다. 이 때 여러가지 표준 함수를 통해 복사할 수 있는데 이 논문에서는 memcpy() 함수가 있다고 가정해 검색한다. 이를 염두에 두고 계속 진행해 보자.

[![](http://dandylife.net/blog/wp-content/uploads/2017/09/step2.png)](http://dandylife.net/blog/archives/699/step2)

**[Step 2] ENCLU 명령어 찾기**: 위 그림에서는 ENCLU 명령어를 찾기 위한 payload를 생성해 확인하는 과정을 보여준다. 앞서 찾은 gadget은 pop 숫자만 알고 있을 뿐 어떤 레지스터에 값이 있는지는 암호화되어 알 수 없다고 했다. 이제 발견한 gadget의 주소와 0x4를 pop 수만큼 넣어 payload를 생성한다. 이렇게 하는 이유는 바로 ENCLU의 특성을 악용(?)하려는 데 있다. gadget 후보군 중 0x4 값을 rax 레지스터에 넣는 명령어가 하나라도 있다면, ENCLU의 EEXIT leaf function이 수행되어 untrusted 구역으로 빠져나오게 될 것이다. EEXIT_handler에서 에러를 탐지하는데 쓰인 조건문을 보자. (PF_PROT|PF_USER|PF_INSTR)는 각각 할당되지 않은 메모리, 사용자 공간의 메모리, 그리고 실행 중 fault가 났을 경우 마지막 두 바이트 ax가 0x4라는 조건이다. 이를 통해 이제 어떤 레지스터가 사용되었는지 모두 복호화할 수 있다!! 여기서 눈여겨 볼 점은 ENCLU EEXIT은 빠져나올 때 레지스터 값을 삭제하지 않는다는 점이다.  실제 인텔 매뉴얼에서도 레지스터 내에서 secret 정보가 있을 경우 이를 삭제하는 역할은 enclave 소프트웨어의 책임이라고 명시하고 있다.

[![](http://dandylife.net/blog/wp-content/uploads/2017/09/step3.png)](http://dandylife.net/blog/archives/699/step3)

**[Step 3] memcpy() 명령어 찾기**: 마지막으로 memcpy() 함수의 주소를 찾는다. 이 단계의 핵심은 enclave에서 중요한 데이터를 가져오거나(copy) 원하는 데이터를 enclave 내부로 주입(inject)하기 위함이다. memcpy()의 인자는 destination, source, length이므로 앞서 찾은 gadget을 이용해 마지막으로 적절하게 payload를 구성한다. (1)에서 공격자가 만든 application에서 fuzzing을 통해 알게 된 버퍼오버플로우를 발생시키고, gadget chain을 따라 (2), (3)과 같이 진행되다 마침내 memcpy() 함수를 만나게 되면 untrusted 메모리 영역의 destination에 값이 바뀌게 될 것이고 이를 통해 복사가 이루어졌음을 인지할 수 있다.

요약하면 Hacking in Darkness에서 사용한 gadget 종류는 크게 세 가지다. 우선 pop과 ret으로만 구성된 gadget, ENCLU gadget, 그리고 memcpy() gadget으로 이 셋은 별도로 찾아낸 후 enclave 내의 데이터를 copy-in / copy-out하는 방식으로 통제할 수 있다는 점이 공격의 핵심이다. 이를 통해 어떤 결과를 초래할 수 있는지 그리고 해결방안은 무엇인지는 직접 논문에서 확인해 보자. 참고로 Enclave 주소 시작점은 OS를 장악하고 있는 공격자가 ASLR 스위치를 끔으로써 간단히 알아낼 수 있다. 마지막으로 이 모든 작업과정을 저자가 직접 유튜브에 공개했으니 다음 링크를 참고해서 감상해 보자. :)

**References**

[Hacking in Darkness at Technical Session in USENIX 2017  
](https://www.usenix.org/conference/usenixsecurity17/technical-sessions/presentation/lee-jaehyuk)[Intel Software Guard Extensions Programming Reference  
](https://software.intel.com/sites/default/files/managed/48/88/329298-002.pdf)[www.youtube.com/watch?v=hyuZFf3QxvM](http://www.youtube.com/watch?v=hyuZFf3QxvM)
