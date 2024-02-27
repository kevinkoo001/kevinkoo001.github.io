---
author: kevinkoo001@gmail.com
comments: false
date: 2014-08-25 05:38:40+00:00
layout: post
link: http://dandylife.net/blog/archives/274
slug: attacks-against-ssltls-beastcrimebreach
title: Attacks against SSL/TLS - BEAST/CRIME/BREACH
wordpress_id: 274
categories:
- Attack &amp; Defense, Cyber Warfare
- Network Security
tags:
- Attack
- network
- Protocol
- SSL
- TLS
---

 **(a) BEAST (Browser Exploit Against SSL/TLS)**



	
  *  By Juliano Rizzo and Thai Duong, 2011 @ ekoparty Security Conference in Buenos Aires, Argentina

	
  *  Decrypts secure cookies against CBC mode (i.e AES or 3DES) in TLSv1


**<References>
**Demo and description: [http://vnhacker.blogspot.com/2011/09/beast.html](http://vnhacker.blogspot.com/2011/09/beast.html)
Paper: [http://packetstormsecurity.com/files/download/105499/Beast-SSL.rar/
](http://packetstormsecurity.com/files/download/105499/Beast-SSL.rar/)Proof of Concept with javascript: [http://erlend.oftedal.no/blog/beast/](http://erlend.oftedal.no/blog/beast/)

**(b) CRIME (Compression Ratio Info-leak Made Easy)**



	
  * By Juliano Rizzo and Thai Duong, 2012 @ ekoparty Security Conference in Buenos Aires, Argentina

	
  * leverages compression side-channel, recovers the HTTP request headers

	
  * Injects partial chosen plaintext (CPA) into a victim’s requests + measures the size of encrypted traffic

	
  * HTTP-level compression: gzip (RFC 1952), defalte (RFC 1951)

	
  * Mitigated by disabling TLS/SPDY compression


**<References>**
1. Wiki: [http://en.wikipedia.org/wiki/CRIME_(security_exploit)
](http://en.wikipedia.org/wiki/CRIME_(security_exploit))2. Tor and BEAST: [https://blog.torproject.org/blog/tor-and-beast-ssl-attack
](https://blog.torproject.org/blog/tor-and-beast-ssl-attack)3. Schneier's Article: [https://www.schneier.com/blog/archives/2011/09/man-in-the-midd_4.html
](https://www.schneier.com/blog/archives/2011/09/man-in-the-midd_4.html)4. Generic attacks with compression: [http://www.iacr.org/cryptodb/archive/2002/FSE/3091/3091.pdf](http://www.iacr.org/cryptodb/archive/2002/FSE/3091/3091.pdf)

**(c) BREACH (Browser Reconnaissance and Exfiltration via Adaptive Compression of Hypertext)**



	
  * By Angelo Prado, Neal Harris, and Yoel Gluck, 2013 @ Blackhat 2013

	
  * CVE-2013-3587

	
  * leverages compression, takes advantage of HTTP responses

	
  * Mitigated by:Disabling HTTP compression
- Separating secrets from user input
- Randomizing secrets per request
- Masking secrets (effectively randomizing by XORing with a random secret per request)
- Protecting vulnerable pages with CSRF
- Length hiding (by adding random number of bytes to the responses)
- Rate-limiting the requests


**<References>**

1. Wiki: [http://en.wikipedia.org/wiki/BREACH_(security_exploit)
](http://en.wikipedia.org/wiki/BREACH_(security_exploit))2. Paper: [http://breachattack.com/resources/BREACH%20-%20SSL,%20gone%20in%2030%20seconds.pdf
](http://breachattack.com/resources/BREACH%20-%20SSL,%20gone%20in%2030%20seconds.pdf)3. PPT: [http://breachattack.com/resources/BREACH%20-%20BH%202013%20-%20PRESENTATION.pdf
](http://breachattack.com/resources/BREACH%20-%20BH%202013%20-%20PRESENTATION.pdf)4. Source Code: [https://github.com/nealharris/BREACH](https://github.com/nealharris/BREACH)
