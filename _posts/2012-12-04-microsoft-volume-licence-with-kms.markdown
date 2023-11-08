---
author: kevinkoo001@gmail.com
comments: true
date: 2012-12-04 23:52:31+00:00
layout: post
link: http://dandylife.net/blog/archives/61
slug: microsoft-volume-licence-with-kms
title: Microsoft Volume Licence with KMS
wordpress_id: 61
categories:
- Digital Forensic &amp; Investigation
- Miscellaneous Stuff
tags:
- KMS
- Microsoft
- volume license
---

마이크로소프트는 Vista 버전부터 기관에서 복수 사용자를 인증하는 볼륨 라이선스를 MAK(Multiple Activation Keys) 이나 KMS(Key Management Server) 방식으로 제공한다. KMS를 통한 방식은 키 관리 서버와 주기적으로 통신하며 180일마다 재확인 절차를 거친다. 다만 MAK으로 인증한 제품의 경우 KMS로 재인증을 수행하지 않는다. 상세한 내용은 아래 링크를 참조하자.

http://en.wikipedia.org/wiki/Volume_license_key
http://www.microsoft.com/Licensing/existing-customers/product-activation.aspx

오피스 2010 버전 정품을 KMS 방식으로 인증하는 방식을 한 번 지켜봤다.

**1. 참조 파일과 레지스트리 위치 (Files and Registry keys for KMS)**

키 인증에 관련된 파일은 아래 디렉토리와 레지스트리를 참조한다.
The key management refers to the directory and the registry location below.

**C:\Program Files\Common Files\Microsoft Shared\OfficeSoftwareProtectionPlatform**
**HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\OfficeSoftwareProtectionPlatform**

**2. 절차 (Procedures)**

(1) 먼저 아래 디렉토리의 vbs를 수행하여 key server를 설정해 준다.
First of all, key server should be identified by executing vbs in the directory below.

C:\Program Files\Microsoft Office\Office14>cscript ospp.vbs /sethst:[_key.management.server_]

위 서버는 아래 registry 위치에 저장된다. 세션ID 또한 존재하는 것으로 보아 일정기간 동안 유효한 것으로 보인다.
The key server value is stored in the specific registry location as follow. There is a ServiceSessionId, which makes using product valid.

**HKLM\SOFTWARE\Microsoft\OfficeSoftwareProtectionPlatform\KeyManagementServiceName**
**HKLM\SOFTWARE\Microsoft\OfficeSoftwareProtectionPlatform\ServiceSessionId**

MS Office 2010이 사용하는 모듈 ID와 레지스트리 디렉토리는 다음과 같다. 해당 모듈을 찾을 수 있다면 MS Office 2010 사용자이거나 사용했던 적이 있다고 판단할 수 있다.
The moduleID and its identifying location might inform investigators that the user has been used MS Office 2010.

moduleID: ba38975c-7786-44bc-b924-147c77920328
**HKLM\SOFTWARE\Microsoft\OfficeSoftwareProtectionPlatform\Plugins\Modules\ba38975c-7786-44bc-b924-147c77920328**
**
**(2) 아래 명령어를 통해 서버로부터 key를 activation한다. 아래 결과는 일부를 변조한 것이다.
The script activates the key from the KMS server. Note that the result has been altered for security reasons.

C:\Program Files\Microsoft Office\Office14>cscript ospp.vbs /act
Microsoft (R) Windows Script Host 버전 5.7
Copyright (C) Microsoft Corporation 1996-2001. All rights reserved.

---Processing--------------------------
---------------------------------------
Installed product key detected - attempting to activate the following product:
*** ID: ****7360-8c5f-417c-9d61-816******e0c
LICENSE NAME: Office 14, OfficeProPlus-KMS_Client edition
LICENSE DESCRIPTION: Office 14, VOLUME_KMSCLIENT channel
Last 5 characters of installed product key: J3AT1
<Product activation successful>
---------------------------------------
---------------------------------------
---Exiting-----------------------------

**3. 흔적 (The artifacts)**

해당 스크립트가 실행되는 동안 시스템의 변화를 살펴봤다.

(1) **이벤트뷰어 (Event viewer)**에서 다음과 같은 정보를 확인할 수 있었다.
하나는 클라이언트에서 활성화 요청을 하고 다른 하나는 서버에서 활성과 결과를 보여준다.
The event viewer recorded an activation results: one for request from client, and the other for response from KMS.


The client has sent an activation request to the key management service machine.




Info:




0x00000000, 0x00000000, _key.management.server:_1688, .......-b604-4b8c-9a48-99d4348fcf37, 2012/12/04 07:06, 0, 2, 972, 6f345b60-8a5c-427c-....-836a982589dc, 5





The client has processed an activation response from the key management service machine.
Info:
0x00000000, 0x00000000, 1, 0, 10, 120, 10080, 2012/12/04 07:06





KMS 서버는 주로 1688 포트를 사용하고 있으며, 외부에서 접근할 수 있도록 설정되어 있을 경우 어떻게 될까 궁금해서 시도해 봤으나 포트가 닫힌 탓인지 product key가 맞지 않아 아래 에러 코드와 함께 불가했다.




The KMS server use port 1688/tcp. I gave a try to switch a different server, but failed for some reasons with errors.


ERROR CODE: 0xC004F074
ERROR DESCRIPTION: The Software Licensing Service reported that the computer cou
ld not be activated. No Key Management Service (KMS) could be contacted. Please
see the Application Event Log for additional information.
To view the activation event history run: cscript ospp.vbs /dhistorykms


수 분 후 활성화가 완료되면 아래와 같이 상태 체크를 하게 된다.




In several minutes, the service checked the licensing status.





The Software Protection service has completed licensing status check.
Application Id=59a52881-a989-479d-af46-f275c6370663
Licensing Status=
1: 191301d3-..-..-b0c7-d7..9e3, 1, 0 [(0 [0xC004F014, 0, 0], [(?)(?)(?)(?)(?)(?)])(1 )(2 )]
2: 6f327760-..-..-9b61-83..70c, 1, 1 [(0 [0x00000000, 1, 0], [(?)(?)( 1 0x00000000 30 0 msft:rm/algorithm/volume/1.0 0x00000000 259192)(?)(?)(?)])(1 )(2 )]
3: fdf3ecb9-b56f-43b2-a9b8-1b48b6bae1a7, 1, 0 [(0 [0xC004F014, 0, 0], [(?)(?)(?)(?)(?)(?)])(1 )(2 )]

Application 이벤트 ID 1003를 MS에서 직접 조회해 보았으나 예상대로 별 내용을 알려주지 않는다.
The Microsoft does not give any information details about application event ID 1003, as expected.


[![](http://2.bp.blogspot.com/-J15cbDz2OJA/UL3CIWhVrzI/AAAAAAAAAGU/AaqeMJYQ-j4/s1600/event1003.JPG)](http://2.bp.blogspot.com/-J15cbDz2OJA/UL3CIWhVrzI/AAAAAAAAAGU/AaqeMJYQ-j4/s1600/event1003.JPG)




(2) **네트워크 패킷 (network packets)**은 요청시 260 바이트, 응답시 176 바이트의 stub 데이터를 포함하는데 아마 activation에 필요한 최소한의 데이터만 주고받는 것으로 보인다.  The network packets containing stub data for key activation was observed: 260 bytes for request and 176 bytes for response.





[![](http://2.bp.blogspot.com/-JBhAmBsdk9c/UL3C6PTyEpI/AAAAAAAAAGc/mBDtZTsz5SY/s1600/npacket.JPG)](http://2.bp.blogspot.com/-JBhAmBsdk9c/UL3C6PTyEpI/AAAAAAAAAGc/mBDtZTsz5SY/s1600/npacket.JPG)










(3) 시스템 변경사항 (system alteration) :스크립트 실행 당시 파일을 추적해 본 결과 흥미롭게도 C:\WINDOWS\AppPatch\sysmain.sdb라는 파일 접근, 오픈 등 40여 차례의 오퍼레이션이 일어났으나 어떤 작업인지는 알아낼 수 없었다. 키관리 서버에서 세션키를 수립하면 레지스트리 아래 위치에 저장되는 것으로 보인다.







While executing the cscript, the operation was found to access and/or open the file, C:\WINDOWS\AppPatch\sysmain.sdb. The session key seems to be stored in particular registry location.







**HKLM\SOFTWARE\Microsoft\OfficeSoftwareProtectionPlatform\Plugins\Modules\ba38975c-7786-44bc-b924-147c77920328**




[![](http://1.bp.blogspot.com/-qllLxgRV_ws/UL3GgCauzyI/AAAAAAAAAGs/0dvpPEO_K8g/s1600/registry.JPG)](http://1.bp.blogspot.com/-qllLxgRV_ws/UL3GgCauzyI/AAAAAAAAAGs/0dvpPEO_K8g/s1600/registry.JPG)



