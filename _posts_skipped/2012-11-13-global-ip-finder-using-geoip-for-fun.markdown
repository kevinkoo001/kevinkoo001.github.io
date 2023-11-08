---
author: kevinkoo001@gmail.com
comments: true
date: 2012-11-13 16:51:39+00:00
layout: post
link: http://dandylife.net/blog/archives/55
slug: global-ip-finder-using-geoip-for-fun
title: Global IP Finder using GeoIP for fun
wordpress_id: 55
categories:
- Attack &amp; Defense, Cyber Warfare
- Miscellaneous Stuff
tags:
- IP Finder
- maxmind
---

위치 추적은 위성을 이용하여 과거 군사용으로 사용됐다. 70년대 후반 GPS(Global Positioning System)의 등장으로 삼각측량법에 기반해 위성의 도움으로 정확한 위치를 추적할 수 있게 되었고 오늘날 민간인이 실시간으로 사용하기까지 그리 오래 걸리지 않았다. 인터넷에서 위치는 IP 주소로 확인할 수 있는데 이에 기반한 물리적 주소를 알 수 있을까?

IPv4에 정의한 프로토콜은 국제 표준화 기구인 IETF(Internet Engineering Task Force)에서 1981년 공표한 문서 RFC791를 보면 ([http://www.ietf.org/rfc/rfc791.txt](http://www.ietf.org/rfc/rfc791.txt)) 확인할 수 있다. 헤더에서 출발지와 목적지 IP는 각각 32비트로 정의해 이론적으로 2^32=4,294,947,296개를 사용할 수 있으나 실제는 그보다 더 적은 수의 IP만 사용 가능하다. 이는 Special Use IPv4 Addresses라는 RFC5735([http://](http://www.blogger.com/goog_41987409)[tools.ietf.org/html/rfc5735](http://tools.ietf.org/html/rfc5735))와 RFC1918([http://tools.ietf.org/html/rfc1918](http://tools.ietf.org/html/rfc1918))에서 확인할 수 있다. 또한 이 IP는 IANA(Internet Assigned Numbers Authority, [http://www.iana.org](http://www.iana.org/))라는 비영리기구에서 관할한다.

Tracing location has been mainly used for the military operation purpose. In the late 70's, the advent of GPS, Global Positioning System, allows people to accurately trace targets with the help of a satellite. Today civilians are permitted to be of use for their own interests. By checking IP(Internet Protocol) address, one is able to recognize the other on the Internet. Then how are we aware of the physical location based on this information, IP address? Well, RFC791 has well defined and explained how it works. It allocated 32 bytes in source and destination IP version 4, which means there are approximately four billion IPs available theoretically. However standard organization has predefined some IPs for special use.(Refer to RFC5735 and RFC1918). The IANA(Internet Assigned Numbers Authority) has administered the entire block of IP addresses.

오늘날 가장 대중적으로 이용되는 방식은 Maxmind사 ([http://www.maxmind.com](http://www.maxmind.com/))의 GeoIP라는 데이터베이스를 사용해서 물리적 위치를 찾는다. 해당 웹사이트에 따르면 국가기반은 99.8%, 주 수준은 90%, 도시 수준으로는 83%의 정확도를 가진다고 한다. 하지만 구글 어스를 통해 검색하면 훨씬 정확한 위치를 얻어낼 수 있음을 보면 매우 상세한 DB를 가지고 있으리라 추정된다. 2012년 2월 기준으로 데이터베이스에 약 15만개의 국가 수준의 IP 블럭을 소유하고 있고 전체 IP수는 34억 3100만여개, 등록국가는 248개이다. ([http://www.maxmind.com/app/techinfo](http://www.maxmind.com/app/techinfo)) 현재 시점에서 당연히 또 변화가 있을 것이다. 다음 주소에서 무료 CSV 파일로 제공하니 궁금한 사람은 직접 다운받아 보도록 하자.

[http://geolite.maxmind.com/download/geoip/database/GeoIPCountryCSV.zip](http://geolite.maxmind.com/download/geoip/database/GeoIPCountryCSV.zip)
[http://geolite.maxmind.com/download/geoip/database/GeoIPv6.csv.gz](http://geolite.maxmind.com/download/geoip/database/GeoIPv6.csv.gz)
[http://geolite.maxmind.com/download/geoip/database/GeoLiteCity_CSV/GeoLiteCity_20120207.zip](http://geolite.maxmind.com/download/geoip/database/GeoLiteCity_CSV/GeoLiteCity_20120207.zip)





Today, the GeoIP database from Maxmind helps us to look for the location. According to the official website, it provides 99.8% accuracy on a country level, 90% on a state level, and 83% for cities in the US within a 25 mile. As of Feb 2012,the total number of IP Blocks on the country level (Database records) is almost 150 thousand, and the total number of IP addresses are 3.43 billions from 248 countries. You can download the database on the country level with free of charge. Note that A1 stands anonymous proxy entries and A2 stands ISPs offering the Internet through satellites. Using these information, ipinfodb provides service to return IP details in XML, JSON format with purchased key. Another API provides block IPs by countries or fraud detection. Below is an example.







여기서 눈여겨 볼 것은 A1으로 분류되는 게 있는데 이는 익명 프락시가 사용하는 IP이며 A2는 ISP와 같은 위성을 통한 IP를 의미한다. ipinfodb와([http://ipinfodb.com](http://ipinfodb.com/)) 같이 라이선스를 구입하면 Perl, ASP, PHP 등의 언어로 IP에 대한 정보를 반환해 주는 Web Service도 제공해 주는 사이트도 있다. 이 IP 위치 API를 이용해 다음과 같이 XML, JSON과 같은 표준방식으로 반환해 준다. 또한 국가별 Block IP가 어디인지 Fraud 탐지 등을 해 주는 API 서비스도 있다.








	
  * XML API: http://api.ipinfodb.com/v3/ip-city/?key=<key>&ip=74.125.45.100

	
  *  JSON API: http://api.ipinfodb.com/v3/ip-city/?key=<key>&ip=74.125.45.100&format=json{ "statusCode" : "OK","statusMessage" : "","ipAddress" : "74.125.45.100", "countryCode" : "US", "countryName" : "UNITED STATES", "regionName" : "GEORGIA", "cityName" : "ATLANTA","zipCode" : "30301", "latitude" : "33.809", "longitude" : -84.3548", "timeZone" : "-05:00“ }





GeoIP 역시 다음과 같은 언어의 API를 제공한다.



	
  * C Library

	
  * Perl Module

	
  * PHP Module

	
  * Apache Module (mod_geoip)

	
  * Java Class

	
  * Python Class

	
  * C# Class

	
  * Ruby Module

	
  * MS COM Object?(ASP, ColdFusion, Pascal, PHP, Perl, Python, and Visual Basic code)

	
  * VB.NET?(Only works with GeoIP Country)

	
  * Pascal

	
  * JavaScript


C와 Python만으로 테스트를 한 번 해 봤다. 파이선의 경우 지도에 나타낼 수 있는 라이브러리를 이용하여 지점을 표기할 수 있도록 응용할 수 있다.

Here are a couple of examples for installations and test outputs using GeoIP API with C library and Python.

**A. C 라이브러리를 이용한 GeoIP API (GeoIP APIs with C Library)**





(1) 설치 (Installation)
# ./configure
# make; make check; make install


(2) 컴파일 (Compilation )
# gcc -lGeoIP getip.c –o getip

(3) 사용법 (Usage)
# include <GeoIP.h>
int main (int argc, char *argv[]) {
GeoIP* gi;
gi = GeoIP_new(GEOIP_STANDARD);
printf("code %s\n“,  GeoIP_country_code_by_name(gi, "yahoo.com"));
}





**B. Python을 이용한 GeoIP API (GeoIP APIs with Python (Pygeoip) )**










(1) 설치 (Installation)




    # wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity_CSV/GeoLiteCity_20120207.zip




    # unzip -d GeoLiteCity_20120207.zip




    # wget http://pygeoip.googlecode.com/files/pygeoip-0.2.2.tar.gz




    # tar zxvf pygeoip-0.2.2.tar.gz




    # cd pygeoip-0.2.2




    # python setup.py build




    # python setup.py install







(2) 테스트 (Test)







    # python




    >>> import pygeoip




    >>> gip = pygeoip.GeoIP(‘GeoLiteCity.dat’)




    >>> rec = gip.record_by_name(‘yahoo.com’)




    >>> for key,val in rec.items():




    ... print “%s: %s” % (key,val)







(3) 결과 (Output)













   city: Sunnyvale




   region_name: CA




   area_code: 408




   longitude: -122.0074




   country_code3: USA




   latitude: 37.4249




   postal_code: 94089




   dma_code: 807




   country_code: US




   country_name: United States







**C. Matplotlib를 이용해 지도에 나타내기 (Generating map with matplotlib)**













(1) 설치 (Installation)




    [http://sourceforge.net/projects/matplotlib/files/matplotlib-toolkits/basemap-0.99.4/basemap-0.99.4.tar.gz](http://sourceforge.net/projects/matplotlib/files/matplotlib-toolkits/basemap-0.99.4/basemap-0.99.4.tar.gz)




    # apt-get install python-tk python-numpy python-matplotlib python-dev




    # tar -xvzf basemap-0.99.4.tar.gz




    # cd basemap-0.99.4/geos-2.2.3




    # ./configure; make; make install




    # cd ..




    # python setup.py build




    # python setup.py install










(2) 테스트 (Test)




  # python mapper.py –a 112.213.89.30,116.193.83.147,119.70.227.138,121.88.250.247,124.217.218.6







(3) 결과 (Output)







[![](http://1.bp.blogspot.com/-uIQvbjKf73Q/UKJVGVVKGGI/AAAAAAAAAF0/8Foh_RrPan0/s320/mapper.jpg)](http://1.bp.blogspot.com/-uIQvbjKf73Q/UKJVGVVKGGI/AAAAAAAAAF0/8Foh_RrPan0/s1600/mapper.jpg)













**D. Excel에서 GeoIP를 이용한 IP Finder 제작 (Global IP Finder using GeoIP)**







이전에 삽질을 통해 이 정보로 국가를 바로 찾을 수 있게끔 Excel로 만들어 본 적이 있다. 이 정도 IP 정보야 인터넷에서 쉽게 바로 찾아볼 수도 있겠지만 이 Form을 이용하면 조직 내 IP 정보를 리스트화해서 매우 편리하게 이용할 수 있을 것이다. VB의 IPSubnetCalc 부분은 Marcus Mansfield가 작성한 코드를 가져와 사용했다. macro를 활성화해야 정상동작한다. 15만여개의 entry가 있으므로 상당히 사이즈가 크다. 7-zip으로 압축했고 이를 풀면 40MB가 넘으니 열리는 데도 상당한 시간이 걸릴 것이다. 아무쪼록 유용하게 사용하길 바란다. 아래는 classless로 구성한 netid와 hostid이며 이를 기반으로 만들었다. ([Download](http://dandylife.net/docs/Country_Finder@Global_IP.7z))







[![](http://2.bp.blogspot.com/-rsnAhOednxc/UKJc4c_-znI/AAAAAAAAAGE/reJHA6Cc0B4/s1600/classlessips.jpg)](http://2.bp.blogspot.com/-rsnAhOednxc/UKJc4c_-znI/AAAAAAAAAGE/reJHA6Cc0B4/s1600/classlessips.jpg)




 




I have made "Global IP Finder using GeoIP" just for fun. You may want to search for IP information on the Internet. However, this could be useful when you need to dig IP information based on your organization. The function, IPSubnetCalc is created by Marcus Mansfield. Download the link above and uncompress it. (7-zip compression)



