---
author: kevinkoo001@gmail.com
comments: true
date: 2012-08-08 16:44:20+00:00
layout: post
link: http://dandylife.net/blog/archives/37
slug: anonymizing-activities-tor-the-onion-routing
title: 'Anonymizing Activities: Tor (The Onion Routing)'
wordpress_id: 37
categories:
- Anonymity
- Network Security
tags:
- anonymity
- onion routing
- tor
---

사람들이 오프라인 상에서 자신만의 공간을 추구하듯이 온라인 상에서도 익명성을 요구하는 노력이 진행되었는데 대표적인 결과물이 tor(The Onion Routing)다. 토르는 인터넷 상에서 프라이버시와 보안을 향상시킬 수 있는 가상 터널 네트워크라고 정의하고 있다. ([https://www.torproject.org/about/overview.html.en#overview](https://www.torproject.org/about/overview.html.en#overview))



	
  * 동작원리: 각 노드는 자신만의 키 쌍을 가지고 라우터 역할을 하는데 공개키로 라우팅 정보를 암호화하여 다음 노드는 이전 경로를 알 수 없도록 함

	
  * 마지막 노드(exit node)에서 데이터를 볼 수 있는데 이를 악용하는 경우도 있음

	
  * 익명성은 강화되지만 속도가 느리다는 단점이 있음

	
  * SOCKS와 HTTPS를 지원함




간단히 정리한 내용은 아래 링크를 참조하기 바란다.
[Anonymizing Activities: Tor(The Onion Routing)](http://dandylife.net/blog/wp-content/uploads/2013/07/Anonymizing-Activities.pdf)

대표적으로 구현한 소프트웨어는 다음과 같다.





	
  * [Tor Browser Bundle for Windows](https://www.torproject.org/download/download-easy.html.en)

	
  * [Advanced Onion Router](http://sourceforge.net/projects/advtor/)


오래 전 Bruce Schneier은 자신의 블로그에서 관련된 공격(MITM)에 대해 논의된 바 있으며 공식 웹사이트에서도 익명성에 대한 공격 가능성을 언급한다.

	
  * [http://www.schneier.com/blog/archives/2007/12/maninthemiddle.html](http://www.schneier.com/blog/archives/2007/12/maninthemiddle.html)

	
  * [https://blog.torproject.org/running-exit-node](https://blog.torproject.org/running-exit-node)


또한 tor 경유지 노드 또는 마지막 노드 현황은 아래 링크에서 찾을 수 있으니 참고하자.

	
  * [http://torstatus.blutmagie.de/](http://torstatus.blutmagie.de/)

	
  * [http://proxy.org/tor.shtml](http://proxy.org/tor.shtml)




As people pursue their own privacy at offline, many attempts have been made to keep anonymity online as well. Tor(The Onion Routing) is one of them. Tor is defined as "a network of virtual tunnels that allows people and groups to improve their privacy and security on the Internet." according to the torproject.org website.







Each node has its own public/private key pair, and the next node does not know where the packets come from by encrypting routing information. However make sure that the exit node to the destination could be compromised by malicious user. It is true that tor considerably improves anonymity but the speed of connection would somewhat slow down. It supports SOCKS and HTTPS.



