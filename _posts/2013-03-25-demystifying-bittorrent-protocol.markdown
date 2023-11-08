---
author: kevinkoo001@gmail.com
comments: true
date: 2013-03-25 19:12:01+00:00
layout: post
link: http://dandylife.net/blog/archives/72
slug: demystifying-bittorrent-protocol
title: Demystifying BitTorrent Protocol
wordpress_id: 72
categories:
- Miscellaneous Stuff
- Network Security
tags:
- BitTorrent
- Protocol
- Torrent
---

BitTorrent는 오늘날 개별 파일 공유에 가장 많이 사용하는 대표적인 프로토콜이다. 또한 저작권 측면에서 보면 2010년 이래에 20만 명 이상이 저작권으로 고소당하기도 했던 장본인이기도 하다. 2001년 Bram이라는 친구가 처음 만든 이 프로토콜에 대해 기술적으로 한 번 살펴보자. F-Insight에서 발표한 내용이기도 하니 다음 자료를 참고하자.( [BitTorrent-Protocol](http://dandylife.net/blog/wp-content/uploads/2013/07/BitTorrent-Protocol.pdf) )

Today the BitTorrent protocol is widely used for file sharing privately. Created by Bram Cohen in 2001, it is estimated that there are 150 million active BitTorrent users, approximately 250 million potential ones as of Jan. 2011. You may want to refer to the material which I presented in "[Forensic Insight Seminar](http://forensicinsight.org/)".

**1. 주요용어 (Terminology)**
**
**(Ref) [http://www.bittorrent.com/intl/ko/help/faq/concepts](http://www.bittorrent.com/intl/ko/help/faq/concepts)

**block** a piece of a file when distributed via BitTorrent
**peer** one of a group of clients downloading the same file
**leech **a peer that is downloading while uploading very little, or nothing at all. a.k.a leecher
**scrape** a request to the tracker for information about the statistics of the torrent
**seed** a complete copy of the file being made available for download
**seeder** a peer that is done downloading a file and is now just making it available to others.
**torrent **the instance of a file or group of files being distributed via BitTorrent
**swarm** a group of seeds and peers sharing the same torrent
**tracker** a server that keeps track of the peers and seeds in a swarm


**블록** BitTorrent 를 통해 배포하는 파일 조각




**피어** 동일한 파일을 다운로드하는 클라이언트 그룹원




**리치** 업로드는 거의 하지 않으며 주로 다운로드하는 피어




**스크레이프** torrent 통계 관련 정보를 트래커에게 질의




**시드** 다운로드할 수 있는 전체 복사본 파일




**시더** 다운로드를 완료한 피어 또는 다른 이에게 파일을 공유할 수 있는 피어




**토렌트** BitTorrent를 통해 배포하는 파일 인스턴스




**스웜** 동일한 토렌트를 공유하는 시드 그룹과 피어




**트래커** 스웜에서 피어나 시드를 찾는 서버










**2. 프로토콜 명세 (Protocol Specification)**







(1) **Becoding (Binary Encoding)**: A way to specify the data in a terse format








	
  * 문자열, 숫자, 리스트, 사전을 아래와 같이 정의해서 간단히 표현한다.










[![](http://1.bp.blogspot.com/-hhywGIOWrfs/UU_hynrg3oI/AAAAAAAAAM8/7HuWyJuVDB4/s1600/bencoding.jpg)](http://1.bp.blogspot.com/-hhywGIOWrfs/UU_hynrg3oI/AAAAAAAAAM8/7HuWyJuVDB4/s1600/bencoding.jpg)


(2) **Torrent File Structure**






	
  * 토렌트 파일구조는 아래와 같은데 크게 Info 영역과 Announce 부분으로 나뉜다.

	
  * Info에서 파일 전체 길이, 이름, 조각길이와 각 조각의 20바이트 SHA1 값으로 구성된다.

	
  * 길이 조각(length piece)은 파일 크기에 따라 다양하나 256KB를 기본으로 사용한다.

	
  * 참고로 Torrent 파일을 직접 편집할 수 있는 웹사이트도 존재한다. ([http://torrenteditor.com](http://torrenteditor.com/))







[![](http://3.bp.blogspot.com/-ZVxKbsKz-fo/UU_h1D6Ho7I/AAAAAAAAANQ/Mv-yaZncxYI/s1600/metainfo.jpg)](http://3.bp.blogspot.com/-ZVxKbsKz-fo/UU_h1D6Ho7I/AAAAAAAAANQ/Mv-yaZncxYI/s1600/metainfo.jpg)





<table cellpadding="0" align="center" cellspacing="0" >
<tbody >
<tr >

<td >[![](http://3.bp.blogspot.com/-GMK__S4XfJQ/UU_h1i98U-I/AAAAAAAAANk/hkhnCm1PEKY/s1600/metainfo_eg.jpg)](http://3.bp.blogspot.com/-GMK__S4XfJQ/UU_h1i98U-I/AAAAAAAAANk/hkhnCm1PEKY/s1600/metainfo_eg.jpg)
</td>
</tr>
<tr >

<td >An example of torrent file for each part - d for dictionaries, l for listings, i for integers, s for strings.
</td>
</tr>
</tbody>
</table>





(3) **Tracker HTTP Protocol**








	
  * 아래 표는 Tracker 서버의 Request 영역으로 가장 중요한 부분은 info_hash, peer_id, port, numwant 정보다.

	
  * Port는 BitTorrent를 구현한 Application마다 다르므로 기본 포트가 다를 수 있다.

	
  * 기본적으로 tracker에게 piece가 있는 곳 정보를 50군데 제공받고 해당 peer에게 전송받기 시작한다.

	
  * 아래 그림에서 Tracker Handshaking을 완료하면 조각을 가진 peer가 소유여부(Have)를 알리고 요청(request)한다.







[![](http://4.bp.blogspot.com/-Oquw850FdR4/UU_h1zpBAiI/AAAAAAAAANw/an522JyxU4o/s1600/tracker_req.jpg)](http://4.bp.blogspot.com/-Oquw850FdR4/UU_h1zpBAiI/AAAAAAAAANw/an522JyxU4o/s1600/tracker_req.jpg)







[![](http://3.bp.blogspot.com/-NoCmp08RlvY/UU_h2DO51bI/AAAAAAAAAN0/DQlLFdwHwq8/s1600/tracker_req_resp.jpg)](http://3.bp.blogspot.com/-NoCmp08RlvY/UU_h2DO51bI/AAAAAAAAAN0/DQlLFdwHwq8/s1600/tracker_req_resp.jpg)










**3. 동작방식 (Operation)**







(1) **File Sharing Mechanism**








	
  * 좌측의 Initial Seeder가 파일을 조각(piece)내고 torrent 파일에 위치정보를 포함하여 tracker로 알려준다.

	
  * 우측의 leecher는 해당 파일의 torrent 정보를 통해 tracker에게 요청하고 응답받은 후 패킷을 받는다.

	
  * 이 때 실제는 Performance Issue로 각 piece는 16KB의 sub-piece로 다시 쪼개어 한 번에 다섯 부분만 받는다.

	
  * 아래 그림에서 고양이 Leecher가 다운완료하면 새로운 Leecher인 닭이 또다시 양쪽에서 요청한다.







[![](http://1.bp.blogspot.com/-JfeWIGSv0yo/UU_h1PbyxVI/AAAAAAAAANM/3dioLtaHgWk/s1600/file_sharing_mechanism.jpg)](http://1.bp.blogspot.com/-JfeWIGSv0yo/UU_h1PbyxVI/AAAAAAAAANM/3dioLtaHgWk/s1600/file_sharing_mechanism.jpg)







[![](http://2.bp.blogspot.com/-Y-ujX6g9kOM/UU_h1LnFXwI/AAAAAAAAANU/1yawgZCA-7s/s1600/file_sharing_mechanism2.jpg)](http://2.bp.blogspot.com/-Y-ujX6g9kOM/UU_h1LnFXwI/AAAAAAAAANU/1yawgZCA-7s/s1600/file_sharing_mechanism2.jpg)







(2) **조각 선택 알고리즘 (****Piece Selection Algorithms)**








	
  * Super Seeding (Initial Seeding Mode): Special Case
A peer has nothing to trade initially, so it is Important to get a complete piece ASAP.This makes peer select a random piece of the file and download it.

	
  * Strict Priority: First Priority
This policy keeps the initial bitfield from each peer, and updates it with every “have” message.Then a peer downloads the pieces that appear least frequently in these peer bitfields.

	
  * Rarest First → General rule
BitTorrent determines the pieces that are most rare among your peers, and download those first.It ensures that the most commonly available pieces are left till the end to download.

	
  * Endgame modeAs the completion time closes, bitTorrent requests all peers at the same time.
If the requests seem to pend, then it would be cancelled immediately.



