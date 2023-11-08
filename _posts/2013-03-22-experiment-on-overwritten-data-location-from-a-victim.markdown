---
author: kevinkoo001@gmail.com
comments: true
date: 2013-03-22 17:22:04+00:00
layout: post
link: http://dandylife.net/blog/archives/69
slug: experiment-on-overwritten-data-location-from-a-victim
title: Experiment on overwritten data location from a victim
wordpress_id: 69
categories:
- Attack &amp; Defense, Cyber Warfare
- Digital Forensic &amp; Investigation
- Malware
tags:
- '3.20'
- Malware
---

3.20 대란이라고 일컫을 만큼 파괴력이 강한 악성코드가 금융기관과 방송매체를 강타했다.

많은 전문가와 전문기관에서 이미 악성코드에 관한 분석 레포트를 내놓고 있지만, 정작 어느정도의 피해를 하드디스크에 가했는지에 대한 내용은 없는 것 같아 간단히 포스팅한다. 매우 간단한 실험이지만 데이터 복구에 도움이 되지 않을까 생각한다.

This posting deals with simple analysis on the location of overwritten area for exploited machine. I hope this could be helpful to preform recovery process.

*** 단계 (Steps)**

(1) VMware 가상 이미지 할당 (10기가), 단일 volume
(2) Windows XP SP3 설치
(3) 감염직전 Snapshot 확보
(4) 감염후 자동종료
(5) 개별 이미지 변환
(6) 두 이미지 바이너리 비교분석

(1) Creates virtual image with the size of 10GB virtual disk (VMware)
(2) Installs Windows XP SP3
(3) Takes a snapshot before malware infection
(4) Takes un-bootable image after infection
(5) Converts each image to raw one.
(6) Compares two binaries

*** 결과 (Results)**

(1) 바이너리 비교 시간 때문에 우선 첫 100MB를 비교해 본 결과 덮어쓴 영역이 일정 간격으로 되풀이되고 있었다.
신기하게도 그 간격은 처음부터 0x330D9FFF (816MB)까지는 0x1BD000 (1,780KB)와 0x3F6000 (4,056KB)이 번갈아  가면서 덮어쓰고 있었고 0xFFFFFFFF (4GB)까지는 이 둘을 합한 0x5B3000 (5,836KB)마다 한 번씩 나타났다.  길이는 정확히 0x19000(100KB)를 유지하고 있었다. 하지만 4GB 이후 영역부터 마지막까지는 모두 합쳐 10회 이하로 나타났고 길이도 이보다 짧게 나타났다.

Looking into the first 100MB due to time-consuming comparison, I found there is a constant distance to the next overwritten area. The periodic distance is 0x1BD000(1,780KB) and 0x3F6000(4,056KB) from the  first address to 0x330D9FFF(816MB) but 0x5B3000(5,836KB) in betwen the address 0x5B3000 and
0xffffffff(4GB). The length of repetitive strings "PRINCIPPES" maintains 0x19000(100KB) to be exact.
After that address, the string has seen less than 10 times with shorter length.


[![](http://1.bp.blogspot.com/-BVCpb650Sb0/UUvUeiLrPkI/AAAAAAAAAMI/8VDDWwrodTU/s1600/comparison.jpg)](http://1.bp.blogspot.com/-BVCpb650Sb0/UUvUeiLrPkI/AAAAAAAAAMI/8VDDWwrodTU/s1600/comparison.jpg)



<table cellpadding="0" align="center" cellspacing="0" >
<tbody >
<tr >

<td >[![](http://4.bp.blogspot.com/-8TxpS3Ll4xk/UUvV4Fee8TI/AAAAAAAAAMY/Y-ZuKeZqw9w/s1600/stringlocation.jpg)](http://4.bp.blogspot.com/-8TxpS3Ll4xk/UUvV4Fee8TI/AAAAAAAAAMY/Y-ZuKeZqw9w/s1600/stringlocation.jpg)
</td>
</tr>
<tr >

<td >**The table shows periodic distance is 0x1BD000(1,780KB) and 0x3F6000(4,056KB) in the address range of 0x00000000 ~ 0x330D9FFF (816MB)**
</td>
</tr>
</tbody>
</table>

<table cellpadding="0" align="center" cellspacing="0" >
<tbody >
<tr >

<td >[![](http://4.bp.blogspot.com/-ByTMvkuP2e8/UUvYG3oua1I/AAAAAAAAAMg/-YghEo0im88/s1600/stringlocation2.jpg)](http://4.bp.blogspot.com/-ByTMvkuP2e8/UUvYG3oua1I/AAAAAAAAAMg/-YghEo0im88/s1600/stringlocation2.jpg)
</td>
</tr>
<tr >

<td >**The table shows periodic distance is 0x5B3000(5,836KB) in the address range of 0x330DA000 ~ 0xFFFFFFFF (4GB)**
</td>
</tr>
</tbody>
</table>
(2) 악성코드(MD5: 9263E40D9823AECF9388B64DE34EAE54)는 맨 처음 MBR 영역을 "PRINCIPPES"로 덮어쓴다.  다른 문자열도 존재하나 이번 실험에서는 HITA..는 한 번도 나오지 않았다. MBR 바로 다음은 주소 0x7000에 있는 VBR 영역이다.
The malware(MD5: 9263E40D9823AECF9388B64DE34EAE54) overwrote MBR area with strings "PRINCIPPES"

It did only 480 bytes, while MBR accounted for 512 bytes. Then the first VBR(Volume boot record)
was overwritten,  where is normally located in 0x7000. No area overwritten with "HITA.." was shown.
<table cellpadding="0" align="center" cellspacing="0" >
<tbody >
<tr >

<td >[![](http://4.bp.blogspot.com/-seuna5EmgXw/UUvVPRjRP4I/AAAAAAAAAMQ/taJI1K64WFU/s1600/overwritten+mbr.jpg)](http://4.bp.blogspot.com/-seuna5EmgXw/UUvVPRjRP4I/AAAAAAAAAMQ/taJI1K64WFU/s1600/overwritten+mbr.jpg)
</td>
</tr>
<tr >

<td >**Destructed MBR**
</td>
</tr>
</tbody>
</table>
<table cellpadding="0" align="center" cellspacing="0" >
<tbody >
<tr >

<td >[![](http://1.bp.blogspot.com/-ndAMIYohiMQ/UUvatf8AsHI/AAAAAAAAAMo/4NG2gOKvNNo/s1600/overwritten+data.jpg)](http://1.bp.blogspot.com/-ndAMIYohiMQ/UUvatf8AsHI/AAAAAAAAAMo/4NG2gOKvNNo/s1600/overwritten+data.jpg)
</td>
</tr>
<tr >

<td >**Destructed data**
</td>
</tr>
</tbody>
</table>

