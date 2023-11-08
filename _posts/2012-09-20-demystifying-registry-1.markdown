---
author: kevinkoo001@gmail.com
comments: true
date: 2012-09-20 16:42:36+00:00
layout: post
link: http://dandylife.net/blog/archives/48
slug: demystifying-registry-1
title: Demystifying Registry (1)
wordpress_id: 48
categories:
- Digital Forensic &amp; Investigation
tags:
- digital forensic
- registry
---

Windows 환경에서 수많은 정보의 보고인 Registry를 하나씩 살펴보자.

**1. dd를 이용한 HDD 덤프 (Dumping HDD with dd)**

집에 있던 무척 오래된 8GB짜리 고물 하드디스크가 있길래 덤프해 봤다.
I dumped the image of old hard drive with the size of 8GB at home.

D:\Tools\Digital Forensic\Dump\fau\FAU.x86>dd.exe if=\\.\G: of=C:\Temp\winxp.img
conv=noerror --localwrt
Statistics for logical volume \\?\Volume{fa049b8f-018b-11e2-8062-005056c00008}\
0x082832000 bytes available
0x082832000 bytes free
0x214b0000 bytes total
Volume Name:    \\?\Volume{fa049b8f-018b-11e2-8062-005056c00008}
Volume Label:   KOO
Mount Points:
G:\
Drive Type:             Fixed
Volume Serial Number:           1c62-13ee
Maximum Component Length:       255

Volume Characteristics:
File system preserves case
File system supports Unicode file names
File System:            FAT32
Mounted:                Yes
Clustered:              No
Volume Extents:
Disk Number:     1
Starting Offset: 0x0000000000007e00
Extent Length:   0x0000000201cbb200

Copying \\.\G: to C:\Temp\winxp.img
Output: C:\Temp\winxp.img
8620061184 bytes
8220+1 records in
8220+1 records out
8620061184 bytes written

Succeeded!

이제 X-Ways Forensics 도구를 이용해 Registry를 추출해 보자.


[![](http://2.bp.blogspot.com/-71QFlomS27o/UFng3P4NxFI/AAAAAAAAAFA/kFo8ZTrHk2g/s1600/regfiles.jpg)](http://2.bp.blogspot.com/-71QFlomS27o/UFng3P4NxFI/AAAAAAAAAFA/kFo8ZTrHk2g/s1600/regfiles.jpg)


**2. Registry 알아보기**
**
****(1) 데이터 유형 목록 (Listing of Registry value data types)**
[http://msdn.microsoft.com/en-us/library/windows/desktop/ms724884(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724884(v=vs.85).aspx)
<table >
<tbody >
<tr >
Value
Type
</tr>
<tr >

<td >


REG_BINARY **(3)**

</td>

<td >


이진 데이터

</td>
</tr>
<tr >

<td >


REG_DWORD **(4)**

</td>

<td >


32비트 숫자

</td>
</tr>
<tr >

<td >


REG_DWORD_LITTLE_ENDIAN **(4)**

</td>

<td >


32bit 숫자 (little-endian format)

</td>
</tr>
<tr >

<td >


REG_DWORD_BIG_ENDIAN **(5)**

</td>

<td >


32비트 숫자 (big-endian format)

</td>
</tr>
<tr >

<td >


REG_EXPAND_SZ **(2)**

</td>

<td >


Null로 끝나는 가변 데이터 문자열로 환경변수 참조 ex) "%PATH%"
**ExpandEnvironmentStrings** 함수로 호출

</td>
</tr>
<tr >

<td >


REG_LINK **(6)**

</td>

<td >


Null로 끝나는 Unicode 문자열로 symbolic link 경로 전용
REG_OPTION_CREATE_LINK를 이용한 **RegCreateKeyEx** 함수 호출

</td>
</tr>
<tr >

<td >


REG_MULTI_SZ **(7)**

</td>

<td >


Null, 빈 문자 (\0)로 끝나는 일련의 문자열




ex) _String1_\0_String2_\0_String3_\0_LastString_\0\0




첫 번째 \0은 첫 번째 문자열 종료, 중간 \0은 각 문자열, 마지막 \0은 sequence 종료를 의미함

</td>
</tr>
<tr >

<td >


REG_NONE **(0)**

</td>

<td >


정의되어 있지 않음

</td>
</tr>
<tr >

<td >


REG_SZ **(1)**

</td>

<td >


Null로 끝나는 문자열. Unicode나 ANSI 문자임

</td>
</tr>
</tbody>
</table>
**(2) Registry 경로와 파일 (Registry Paths and Files)**

Registry는 Hive 구조로 되어 있으며 아래 링크에 잘 설명되어 있으니 참조하자.
[http://technet.microsoft.com/en-us/library/cc750583.aspx](http://technet.microsoft.com/en-us/library/cc750583.aspx)

키는 다음 6개로 구성된다.
HKEY_CLASSES_ROOT, HKEY_CURRENT_USER, HKEY_LOCAL_MACHINE, HKEY_USERS, HKEY_CURRENT_CONFIG, HKEY_DYN_DATA
<table >
<tbody >
<tr >
Key
설명
</tr>
<tr >

<td >


HKEY_CLASSES_ROOT

</td>

<td >


HKEY_LOCAL_MACHINE\SOFTWARE\Classes의 Symbolic Link

</td>
</tr>
<tr >

<td >


HKEY_CURRENT_USER

</td>

<td >


HKEY_USERS 사용자 프로파일 Hive 하위 키 Symbolic link

</td>
</tr>
<tr >

<td >


HKEY_LOCAL_MACHINE

</td>

<td >


물리적 hive는 존재하지 않으며 hive 구조의 다른 키를 보유함

</td>
</tr>
<tr >

<td >


HKEY_USERS

</td>

<td >


로그온 계정의 사용자 프로파일 hive를 담고 있는 장소

</td>
</tr>
<tr >

<td >


HKEY_CURRENT_CONFIG

</td>

<td >





현재 하드웨어 정보를 가지고 있는 키 Symbolic link


(HKEY_LOCAL_MACHINE \SYSTEM CurrentControlSet\Control\IDConfigDB\Hardware Profiles 하위)



</td>
</tr>
<tr >

<td >


HKEY_DYN_DATA

</td>

<td >


데이터 탐색 성능을 위한 장소이며 물리적 hive는 존재하지 않음

</td>
</tr>
</tbody>
</table>
레지스트리는 크게 6개의 물리적 파일과 2개의 휘발성 파일이 있다.
There are six physical hive files in hard disk and two volatile files in memory.
<table >
<tbody >
<tr >
Registry Path
File Path
</tr>
<tr >

<td >


HKLM\System

</td>

<td >


%WINDIR%system32\config\SYSTEM

</td>
</tr>
<tr >

<td >


HKLM\SAM

</td>

<td >


%WINDIR%system32\config\SAM

</td>
</tr>
<tr >

<td >


HKLM\Security

</td>

<td >


%WINDIR%system32\config\SECURITY

</td>
</tr>
<tr >

<td >


HKLM\Software

</td>

<td >


%WINDIR%system32\config\SOFTWARE

</td>
</tr>
<tr >

<td >


HKLM\Hardware

</td>

<td >


휘발성 Hive

</td>
</tr>
<tr >

<td >


HKLM\System\Clone

</td>

<td >


휘발성 Hive

</td>
</tr>
<tr >

<td >


HKEY_USERS\User SID

</td>

<td >


사용자 Profile (NTUSER.DAT)
"Document and Settings\User" (WinXP), "Users\User" (Vista 이후)

</td>
</tr>
<tr >

<td >


HKEY_USERS\Default

</td>

<td >


%WINDIR%system32\config\DEFAULT

</td>
</tr>
</tbody>
</table>
**(3) 하이브 구조 (Hive Structures)**





Registry는 파일 시스템에서 블록단위와 마찬가지로 논리적인 Hive 구조를 가지고 있다. Registry의 블록 크기는 4,096바이트(4KB)이며 새로운 데이터가 Hive에 추가될 때 block으로 증가한다. Hive는 다섯 가지의 데이터 유형으로 Key cell, Value cell, Subkey-list cell, Value-list cell, Security-descriptor cell로 구성된다. 특히 아래 보이는 kv, kn은 Signature다. (Little Endian)







Registry has logical hive structure just like blick unit in file system. The size of the block is 4,096 Bytes (4KB). It increments by block size when adding new data. Hive structure has 5 different data types: Key cell, Value cell, Subkey-list cell, Value-list cell, Security-descriptor cell. See the signatures within the excerpt of a raw registry file.







[![](http://4.bp.blogspot.com/-W0xX-7MFrWA/UFnj1jtKOqI/AAAAAAAAAFQ/Qp-Aqj7EjSY/s1600/registry_inside.jpg)](http://4.bp.blogspot.com/-W0xX-7MFrWA/UFnj1jtKOqI/AAAAAAAAAFQ/Qp-Aqj7EjSY/s1600/registry_inside.jpg)



<table >
<tbody >
<tr >
Cell
Data Type
</tr>
<tr >

<td >


Key cell

</td>

<td >


레지스트리 키를 저장하고 있으며 키 노드라고도 함
* Signature: 키-kn, kl-심볼릭 링크
* 키가 최종 업데이트된 timestamp (LastWrite)

</td>
</tr>
<tr >

<td >


Value cell

</td>

<td >


키값과 데이터를 저장하는 셀
*  Signature : kv
* 유형: 상단 레지스트리 유형 참조 (e.g., REG_DWORD, REG_BINARY),
* 값 이름
* 값의 데이터를 담고 있는 셀 인덱스

</td>
</tr>
<tr >

<td >


Subkey-list cell

</td>

<td >


키 셀을 가리키는 일련의 인덱스로 구성

</td>
</tr>
<tr >

<td >


Value-list cell

</td>

<td >


값 셀을 가리키는 일련의 인덱스로 구성

</td>
</tr>
<tr >

<td >


Security-descriptor cell

</td>

<td >


보안 식별자를 가지고 있는 셀
*  Signature: ks



</td>
</tr>
</tbody>
</table>

