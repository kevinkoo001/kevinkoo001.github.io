---
author: kevinkoo001@gmail.com
comments: false
date: 2015-10-28 06:14:58+00:00
layout: post
link: http://dandylife.net/blog/archives/534
slug: internet-censorship
title: Internet Censorship
wordpress_id: 534
categories:
- Network Security
- Security in Society
---

**1. Overview of Internet censorship**

이제 인터넷은 일상 속의 일부가 된 지 꽤 오래다. 가상환경이라는 말이 무색하게 우리 생활에 깊이 관여하면서 지역, 사회, 국가, 시공간을 초월하여 정보를 얻을 수 있는 편리한 도구로 자리잡았다.  하지만 무수히 많은 정보 중 다양한 이유로 (문화, 관례, 관습, 종교, 정치 등) 특정한 정보를 탐탁치 않게 생각하는 세력이 있고 이를 조직, 이익단체, 회사, 정당, 또는 국가차원에서 일반인이 쉽게 접근할 수 없도록 검열(censorship)하기 시작했다.

일반적으로 검열이란 단어, 이미지, 생각 등을 억제하는 그 자체 또는 방법까지 포함하는 의미라고 볼 수 있다. 온라인에서 Censorship은 기술적으로 네트워크 계층 상단에서 종종 이뤄진다. 인터넷 이전에는 유선전화 선로를 물리적으로 단절하거나 신호를 방해하는 형태로(radio jamming이나 broadcast disruption) 가능했었다. 가장 간편한 방법은 인터넷 라인을 끊어버리는 것이지만, 국가 차원의 망을 AS 단위로 완전히 통제하지 않는 이상 IP의 Packet switching 특성 때문에 routing 경로는 다양할 수 밖에 없도록 설계되어 있다.  TCP/IP를 표준으로 사용하는 현대의 censorship은 크게 IP Layer에서 IP를 차단하거나 경로를 강제 우회하는 방식, TCP Layer에서 접속을 방해하는 방식, DNS 프로토콜 동작을 오용하는 방식,  그리고 Application Layer에서 사실상 정보교환의 표준이 된 HTTP 프로토콜에서 URL 접근차단이나 우회 등으로 나눌 수 있다.

**2. Censorship at IP Layer**

IP 계층에서 접속제한은 두 가지로 볼 수 있다.

첫째는 IP 주소자체에 대한 **접근통제 목록 (ACL; Access Control List)**을 router나 방화벽(firewall) 등 가능한 장비에 설정하는 방식이다. Router에서 Blackhole routing 설정도 같은 맥락이다. 이는 상당히 간편한 configuration으로 - [CIDR (Classless Inter-Domain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 등을 통해 - IP mapping이 가능하다는 장점이 있으나, 해당 IP를 일일히 조회해야 하는 번거로움이 있다. 설령 이를 자동화한다 해도 IP 자체 Blocking은 의도치 않게 개별이 아닌 NAT를 사용하는 조직이나 기관 단위 전체가 대상이 되어 버릴 수도 있고, Proxy를 통한 우회도 매우 쉽다. 또한 복수 개의 Host가 하나의 IP를 사용하는 경우에도 원하지 않는 차단이 이뤄질 수 있다. 영국같은 경우 해적 사이트의 경우 Proxy까지 아예 막기 시작했다. ([관련기사](https://torrentfreak.com/uk-isps-quietly-block-lists-of-pirate-bay-proxies-150310/))

둘째는 BGP (Border Gateway Protocol)를 이용해 주변 [AS (Autonomous system)](https://en.wikipedia.org/wiki/Autonomous_system_(Internet))에 **가짜 경로 정보(bogus route)**를 뿌리는(advertise) 방식이다. 비록 의도하지 않았지만, 가장 악명높은 BGP 오용 사례 중 하나는 파키스탄 정부가 소유한 국영통신업체 [PTA (Pakistan Telecommunication Authority)](http://www.pta.gov.pk/)에서 특정 IP를 단속하고자 Youtube가 소유한 한 C Class 단위의 IP 대역을 neighbor AS에 가짜 정보를 흘린 (hijacking) 사고다. 당시 Youtube는 208.65.153.0/22 대역을 소유하고 있었는데, PTA AS (17557)에서 해당 IP대역 24 bit가 본인이 소유하고 있다고 거짓으로 알렸고 (announce), 이 정보를 받은 AS가 인접 AS에 퍼뜨리면서 수 분 내로 전 세계의 Youtube 트래픽 (AS 36561) 이 PTA로 몰리기 시작했다. 물론 PTA는 해당 IP를 블랙홀 처리했기 때문에 당연히 접속히 불가능했다. 아래 동영상을 보면 어떻게 해당 route 정보가 전파되었는지 알 수 있다. 시간대별로 세부 정보는 [여기](http://research.dyn.com/2008/02/pakistan-hijacks-youtube-1/)에서 확인할 수 있다.

[youtube https://www.youtube.com/watch?v=IzLPKuAOe50]

이는 프로토콜 자체로 보면 매우 정상적인 행위인데, 바로 BGP announce는 더 상세한 prefix를 참조 (longest prefix match)하는 규칙을 따르고 있기 때문이다. 또한 놀랍게도 BGP에는 이웃 AS에게 route 정보를 송수신할 때 인증(authentication) 메커니즘이 없다. 이처럼 국지적인 행위가 전세계에 부정적인 영향을 미치는 형태를 '**[부수적인 피해 (Collateral Damage)](https://en.wikipedia.org/wiki/Collateral_damage)**'라고 한다.

**3. Censorship at TCP Layer**

TCP 계층은 **RST 패킷**을 이용하는 방법이 일반적이다. 사실 RST Packet은 정상적인 통신규약의 일부로 해당 연결에서 목적지(destination)가 더 이상의 데이터가 가용하지 않을 때나 출발지(source)에서 데이터를 더 이상 받지 않으려 할 때 보낸다. 실제 전체 flow의 10-15% 정도가 FIN이 아닌 RST으로 접속을 종료(termination)한다고 알려져 있다.

Censorship 용도로 RST을 전송해 접속을 강제로 끊는 경우 설치 위치에 따라 흔히 두 가지 유형으로 나눈다. 하나는 **인라인** (**in-path**) 또는 **경로 내부**에 설치해서 모든 패킷을 직접 관통하게 해서 검열하는 방식이다. 다른 하나는 감청장치(wiretap)를 이용해 패킷을 포워딩(forwarding)한 후 감시하면서 대상(target)이 보일 때 RST을 전송하도록 **경로 외부**에 (**on-path 또는 out-of-path**) 설치하는 방식이다. 전자는 모든 패킷에 직접적으로 관여하므로 패킷을 없애거나 (drop) 주입(injection)함으로써 능동적으로 통제를 하기 쉽다는 장점이 있다. 하지만 단일 실패점 (Single Point of Failure)으로 검열에 실패할 가능성도 존재하고, 트래픽 증가에 따른 확장성(scalability)도 용이하지 않다. 반대로 후자의 경우 간접적으로 관여하므로 훨씬 안정적으로 운영할 수 있고 병렬처리가 가능한 장점이 있는 대신, Censor가 직접 Packet을 제거(removal)하거나 조작/변경 (manipulation 또는 replacement)하기는 불가능하다. 다시 말해 클라이언트는 실제 서버에서 전송한 메시지를 결국 수신하게 된다는 의미다. 경로 외부 설치는 안정적인 운영이라는 장점 때문에 현장에서 IDS (Intrusion Detection System)나 DPI (Deep Packet Inspection)에 많이 사용하고 있는 실정이다.

그럼 On-path 방식을 좀 더 살펴보자. 직접적인 패킷 조작이 불가하므로 이 형태의 censorship이 신뢰할 만한 수준 (reliable)에서 동작한다면, 항상 탐지 가능(detectable)하다. 또한 어떤 방식으로 검열하고 있는지 그 메커니즘까지 알 수 있다. 그 이유는 On-path 검열장비는 결코 상대 (endpoint) 로부터 오는 응답 (response)를 완벽히 재현할(mimic) 수 없기 때문이다. 예를 들어 censor 장비에서 오는 TTL 값은 실제 상대가 송신하는 응답과는 다를 수 밖에 없다. 또한 송신자가 기대하는 다음 패킷의 sequence 번호는 자신이 보낸 번호와 패킷 길이를 더한 값이어야 하지만, censor 장비가 보낸 패킷은 이를 흉내낸다 하더라도 수신자가 최소 두 번을 받게 될 것이다. 패킷 왕복 시간 (RTT, Round Trip Time) 역시 차이가 날 것이다.

여기서 **경쟁조건 (race condition)**이 필히 발생하는데, 하나는 RST 패킷을 받고 난 이후 Data를 받는 경우, 또 하나는 Data를 받고 나서 RST를 받는 경우다. 전자는 수신자 입장에서 이상하게 보일 테고 (out of specification) - 그래도 연결은 끊어진다 - 후자는 이미 Data를 수신했으므로 RST을 무시해 (ignore) 버릴 것이다. 이런 이유로 중국의 검열장비(Great Firewall of China)는 성공율을 높이기 위해 RST을 정확히 세 번 보낸다. Richard는 [Ignoring the Great Firewall of China](http://www.cl.cam.ac.uk/~rnc1/ignoring.pdf)라는 논문에서 클라이언트와 서버 양단에서 RST을 모두 무시할 경우 RST 패킷을 보내는 경우에도 성공적으로 접속할 수 있다는 결과를 내놓았다.

**4. Censorship at DNS**

DNS (Domain Name System) 는 도메인 명(Domain name)을 IP로 변환해 주기 위해 계층적으로 (Hierarchical) 위임하는 구조로 질의할 수 있는 시스템이다. 클라이언트는 Local DNS 서버에 질의를 하는데, 이 서버를 stub resolver 또는 non-authoritative resolver라고 한다. Local DNS는 우선 자신의 cache에 해당 정보가 있는지 검색한 후 없으면 차례로 root DNS, TLD (Top Level Domain) DNS로 질의한 후 결국 해당정보를 가진 최종 authoritative resolver에게 질의해서 응답을 받게 된다.

아래의 경우 구글 DNS (8.8.8.8)에 질의한 결과 Root 'e' 서버와 d.gtld-servers.net라는 TLD 서버로부터 recursive하게 정보를 받아 authoritative server인 ns1.newt.arvixe.com에게 결국 dandylife.net의 IP를 받아오는 모습을 볼 수 있다.
    
    $ dig @8.8.8.8 dandylife.net +trace
    
    ; <<>> DiG 9.7.3 <<>> @8.8.8.8 dandylife.net +trace
    ; (1 server found)
    ;; global options: +cmd
    .                       8060    IN      NS      a.root-servers.net.
    .                       8060    IN      NS      b.root-servers.net.
    .                       8060    IN      NS      c.root-servers.net.
    .                       8060    IN      NS      d.root-servers.net.
    .                       8060    IN      NS      e.root-servers.net.
    .                       8060    IN      NS      f.root-servers.net.
    .                       8060    IN      NS      g.root-servers.net.
    .                       8060    IN      NS      h.root-servers.net.
    .                       8060    IN      NS      i.root-servers.net.
    .                       8060    IN      NS      j.root-servers.net.
    .                       8060    IN      NS      k.root-servers.net.
    .                       8060    IN      NS      l.root-servers.net.
    .                       8060    IN      NS      m.root-servers.net.
    ;; Received 228 bytes from 8.8.8.8#53(8.8.8.8) in 14 ms
    
    net.                    172800  IN      NS      d.gtld-servers.net.
    net.                    172800  IN      NS      a.gtld-servers.net.
    net.                    172800  IN      NS      g.gtld-servers.net.
    net.                    172800  IN      NS      m.gtld-servers.net.
    net.                    172800  IN      NS      b.gtld-servers.net.
    net.                    172800  IN      NS      e.gtld-servers.net.
    net.                    172800  IN      NS      j.gtld-servers.net.
    net.                    172800  IN      NS      i.gtld-servers.net.
    net.                    172800  IN      NS      f.gtld-servers.net.
    net.                    172800  IN      NS      l.gtld-servers.net.
    net.                    172800  IN      NS      c.gtld-servers.net.
    net.                    172800  IN      NS      k.gtld-servers.net.
    net.                    172800  IN      NS      h.gtld-servers.net.
    ;; Received 500 bytes from 192.203.230.10#53(e.root-servers.net) in 107 ms
    
    dandylife.net.          172800  IN      NS      ns1.newt.arvixe.com.
    dandylife.net.          172800  IN      NS      ns2.newt.arvixe.com.
    ;; Received 114 bytes from 192.31.80.30#53(d.gtld-servers.net) in 36 ms
    
    dandylife.net. 86400 IN A 143.95.249.220
    dandylife.net. 86400 IN NS ns1.arvixeshared.com.
    dandylife.net. 86400 IN NS ns2.arvixeshared.com.
    ;; Received 99 bytes from 169.55.246.167#53(ns1.newt.arvixe.com) in 54 ms

여기서 DNS의 문제를 발견할 수 있는데, 바로 매번 질의하기 번거로우므로 일정시간 동안 non-authoritative server의 cache에 저장한다는 점이다. 이 Entry가 잘못되었을 경우 사용자는 Poisoning 공격에 노출된다. 이는 Kaminsky가 2008년 _**DNS Cache poisoning**_이라는 공격을 연구해 발표하면서 정식으로 세상에 알려지게 되었다. 자세한 사항은 [여기](http://unixwiz.net/techtips/iguide-kaminsky-dns-vuln.html)를 참조하도록 하자. 멋진 도식과 함께 세부사항까지 잘 설명되어 있다.

Censorship을 하는 과정에서도 DNS는 쉽게 이를 악용한 표적이 될 수 있다. 국가나 기관이 DNS를 장악 (compromise)할 수 있다면, 이와 같은 poisoning 공격이나 redirection IP를 알려줄 수 있다. TCP RST과 같은 방식으로 on-path에서 DNS injection 또한 가능한데, 이는 일종의 "[Man-on-the-side](https://en.wikipedia.org/wiki/Man-on-the-side_attack)" 공격이라 볼 수 있다. 2012년 SIGCOMM에 익명(anonymous)으로 기고한 한 논문, [ The Collateral Damage of Internet Censorship by DNS Injection](http://www.sigcomm.org/sites/default/files/ccr/papers/2012/July/2317307-2317311.pdf)에서 중국 도처에 있는 39개 AS의 Censorship으로 인해 다른 국가가 얼마나 피해를 입고 있는지에 대한 연구결과를 내놓았다.  중국에 위치한 루트 DNS 서버인 F, I, J 때문에 일부 TLD 서버가 중국의 ISP를 경유할 때 제대로 된 DNS 응답을 받지 못한다는 것이다. 한국도 사실 영향권에 있으며 독일 쪽 서버와 통신할 때 피해를 입고 있음을 알 수 있다.

가짜 DNS 응답을 우회(evade)하는 연구도 있었다. Haixin은 2012년 _**Hold-on**_이라는 기법을 그의 논문에서, [Hold-On: Protecting Against On-Path DNS Poisoning](http://conferences.npl.co.uk/satin/papers/satin2012-Duan.pdf), 소개했다. 아이디어는 DNS 응답을 바로 받아 처리하지 말고 일정시간 동안 기다리라는 것이다. On-path의 경우 실제 DNS 응답이 이후에 도착하게 될 것이므로 정상적인 response을 받고 피해를 최소화할 수 있다. 하지만 이 역시 non-authoritative server가 censorship을 행하는 국가 내부에 있는 경우 cache poisoning에 직접적으로 노출될 수 있으므로 적용할 수 없다.

DNS 위조 여부를 확인할 수 있는 간편한 방법은 censorship이 적용되지 않은 것으로 알려진 DNS 서버에 직접 질의해서 그 결과를 비교해 보는 것이다. [중국 DNS List ](http://public-dns.tk/nameserver/cn.html)중 하나를 선택해 차단되어 있는 것으로 알려진 facebook.com을 테스트해 본 결과 다른 응답을 하고 있음을 알 수 있다. 이 역시 DNS에 인증 메커니즘이 없기 때문에 변조가 용이하다. (인증을 포함한 S-BGP나 DNSSEC은 잘 설계된 프로토콜이지만, motivation이나 overhead 등의 문제로 널리 쓰이고 있지는 않다. 이는 이 글에서 논외로 한다.)
    
    $ nslookup
    > server 220.231.248.68
    Default server: 220.231.248.68
    Address: 220.231.248.68#53
    > facebook.com
    Server:         220.231.248.68
    Address:        220.231.248.68#53
    
    Non-authoritative answer:
    Name:   facebook.com
    Address: 159.106.121.75
    > server 8.8.8.8
    Default server: 8.8.8.8
    Address: 8.8.8.8#53
    > facebook.com
    Server:         8.8.8.8
    Address:        8.8.8.8#53
    
    Non-authoritative answer:
    Name:   facebook.com
    Address: 31.13.69.197

**5. Censorship at HTTP (Application Layer)**

마지막으로 Web protocol의 표준인 HTTP에서 censorship을 알아보자. HTTP의 경우 중간에 Proxy를 이용하는 경우가 대부분이다. 표준구성 또한 앞에서 살펴봤듯이 in-path와 on-path가 존재하며, 이로 인한 장단점 또한 유사하다.

Web proxy는 메커니즘에 따라 크게 **flow terminating** 방식과 **flow rewriting** 방식이 있다. 전자는 Proxy가 완전히 TCP 연결을 대신하는 방식으로 실제 사용자 입장에서 목적지는 proxy인 셈이다. 또한 서버 입장에서 클라이언트는 proxy가 되며, 이 방식은 표준에 가깝게 많이 사용한다. 어떤 경우 HTTP 헤더를 변조하기도 한다. 후자는 censor가 필요한 내용(content)가 발견될 경우에만 변경하는 방식으로 구현, 탐지가 모두 어렵다.  censor 대상에 따라 나눈다면, 완전(complete) proxy와 부분(partial) proxy로 나눌 수 있다. 전자는 모든 트래픽을 다 통과시키는데 반해 후자는 특정 IP에 대해서 rerouting하는 방식이다.

사실 SSL/TLS와 같이 암호화된 패킷 역시 proxy의 인증서를 강제로 신뢰하게 해서 연결을 강요한다면, MITM ([Man-in-the-Middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)) 공격과 같은 형태로 모든 패킷을 평문으로 감시할 수 있다. 사용자 입장에서 브라우저 등에서 목적지 인증서와 맞지 않는다는 경고를 볼 수 있겠지만, 이 또한 브라우저에 강제로 신뢰된 인증서를 설치하는 경우 아무런 제약없이 censor가 가능하다.

HTTP를 통제하기 위해 여러국가에서 관찰할 수 있는 몇 가지 censor 방식이 있다. 첫째, 블랙리스트를 정의하고 사용자가 알기 어렵도록 아무런 응답을 주지 않는 것이다. 둘째, 의도하지 않은 사이트로 redirection하는 방식도 있다. 한국에서도 방송통신 심의 위원회와 사이버 경찰청에서 불법유해 사이트를 정의하고 도박, 북한관련, 성인물 등의 컨텐츠를 담은 사이트의 경우 www.warning.or.kr로 이동시킨다. 놀랍게도 Alexa World Ranking 3000위에 육박할 정도로 잦은 방문(?)을 하고 있다. 셋째, 특정 keyword에 대해서만 반응하는 방식이다. 해당 사이트가 사전에 정의한 키워드를 담고 있는 경우 차단한다.

서구국가에서 검열이 가능한 URL filtering 제품을 다수 개발했는데, 대표적으로 [Websense](http://www.websense.com/), [Bluecoat](https://www.bluecoat.com/), [Netsweeper](http://www.netsweeper.com/), 그리고 맥아피의 [smartfilter ](http://www.mcafee.com/us/products/web-protection.aspx)등이 있다. (맥아피는 인텔에 인수됨) 대규모 회사에 근무한 분이라면 한 번쯤 들어봤을 만한 제품군이다. 중동지역에서는 국가 차원에서 이 제품을 구매해 실제 적용한 상태다. 예멘, 시리아에 Websense와 Bluecoat 제품을 수출했으나 최근 연구결과로 인해 이 사실이 알려지게 되자 해당 회사는 업데이트 지원을 중단한 상태다. 특히 인권 운동 등을 방해하기 위해 사용되었다는 사실이 알려져 관련 문의를 했으나, 비난 등을 우려해서인지 답변을 하지 않는 경우도 있었다고 한다.

**6. Net Neutrality**

지금까지 대부분 Censorship에 관해서 논의했다. 우회 (bypass)하는 방법 또한 다양하지만, 여기서 다루지는 않겠다.

수많은 검열이 현실화되면서 각국에서는 **[망 중립성 (net neutrality)](https://en.wikipedia.org/wiki/Net_neutrality)**에 관해 우려를 표명하기 시작했다. 국가나 ISP에서 모든 데이터를 사용자, 컨텐츠, 사이트, 플랫폼, 애플리케이션, 통신 모드 등에 차별/차등없이 (not discriminately / differentially) 다룰 수 있게 해야 한다는 게 골자다. IP를 발명한 Vint Cerf나 Web의 창시자 Tim Berners Lee도 같은 편에 서 있다. 최근 미국 NSA에서 모든 통신내용을 감청하고 있다는 내용을 폭로한 Edward Snowden도 반대진영에 맞서 싸우고 있으며, 많은 사람들의 호응을 얻고 있다. 아래는 2014년 TED에서 연설한 그의 동영상이다.

[youtube]xxEw98fbsXo[/youtube]

우리나라 또한 인터넷 검열에서 자유롭지 못하다. [여기 Wikipedia](https://en.wikipedia.org/wiki/Censorship_by_country)에  국가별 검열 현황이 잘 나타나 있다. 정치/종교/이해관계/문화/사회 등 여러 가지 이유로 censorship을 시행하고 있지만,  당장 옮고 그름으로 판단하기 어려워 보인다. 마지막으로 Netsweeper에서 제공하는 **[Deny Page Tests](http://denypagetests.netsweeper.com/)**에서 어떤 URL이 Filtering되고 있는지 테스트해 보면 어떨런지.
