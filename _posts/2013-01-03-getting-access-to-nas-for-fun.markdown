---
author: kevinkoo001@gmail.com
comments: true
date: 2013-01-03 14:51:45+00:00
layout: post
link: http://dandylife.net/blog/archives/63
slug: getting-access-to-nas-for-fun
title: Getting access to NAS for fun
wordpress_id: 63
categories:
- Attack &amp; Defense, Cyber Warfare
tags:
- NAS
- root access
---

Using NAS storage made by LG, I needed to get access with root privilege like php.ini modification.
I found it interesting because there has been a discussion about this issue.
Well, I think LG Electronics should be more careful in security when releasing next version.


**1. Overview**

You can download up-to-date NAS (N1T1 here) firmware, whose version is 10119 released in Aug. 27, 2012.
[http://www.lgservice.co.kr/cs_lg/download/SoftwareDownloadMainCmd.laf](http://www.lgservice.co.kr/cs_lg/download/SoftwareDownloadMainCmd.laf)

The 7-zip utility allows you to decompress new version (called New UI), "**firmware-nt1_10119rfke.bin**".
Then you will see 11 extacted files including a single gzipped tar ball, two shell scripts, and files containing values.


 [![](http://4.bp.blogspot.com/-eyfVQyW6-TQ/UOTOTc9fyjI/AAAAAAAAAKw/CTZ6wLvtLwA/s1600/list2.jpg)](http://4.bp.blogspot.com/-eyfVQyW6-TQ/UOTOTc9fyjI/AAAAAAAAAKw/CTZ6wLvtLwA/s1600/list2.jpg)


**
2. Summary**

I largely referred to the following postings:
[http://forum.nas-portal.org/archive/index.php/t-14664.html](http://forum.nas-portal.org/archive/index.php/t-14664.html)
[http://forum.nas-portal.org/archive/index.php/t-14744.html](http://forum.nas-portal.org/archive/index.php/t-14744.html)

To make a long story short, these guys did make use of the flow of installation and add a superuser:



	
  * The firmware.tar.gz file seemed encrypted/password protected.

	
  * The gz compressed file hinted that the ENCRYPTION has METHOD_1 in the container.

	
  * There were shell scripts during before and after installation, named _**preinst.sh**_ and _**postinst.sh**_.

	
  * By adding another user who has root privilege in _postinst.sh_, they could get full access to NAS file system.

	
  * They installed another SSH daemon because SSH was running but only listened to a passkeyfile.

	
  * Since "Old UI" firmware indicated how to decrypt encrypted file, they could still use the same way.


**
3. Details**

**(a) Postinst.sh manipulation**
There is a hidden configuration page to setup telnet. (You can access to this page after login.)
[http://[your-nas-ip]/configuration/network/pop_telnetssh.html](http://[your-nas-ip]/configuration/network/pop_telnetssh.html)

By setting this up, telnet service is available (listening 23/tcp). SSH uses 2020/tcp by default, but it will fail you to sign in with this service due to the reason above. With 7-zip, put additional lines in _**postinst.sh**_ as following. Make sure your editing follows UNIX style, otherwise it will screw up. CR/LF in windows might lead an error.
useradd -o -u -g 0 -m [youraccount]
echo [youraccount]:[yourpassword] | chpasswd

This setting changes original permission, however it works well.

**(b) Upgrade**
Login with administrator permission, and go configuration menu.
Click "Firmware update" in System section. Upgrade firmware manually, uploading your box.


[![](http://1.bp.blogspot.com/-gKdSRiYXbEM/UOTORQ8LQdI/AAAAAAAAAKo/3BAoZsWVN4M/s1600/upgrade.jpg)](http://1.bp.blogspot.com/-gKdSRiYXbEM/UOTORQ8LQdI/AAAAAAAAAKo/3BAoZsWVN4M/s1600/upgrade.jpg)


**(c) Another SSH installation: _dropbear_**
You should install another SSH if you need SSH connection for further connection.
For more information: [https://matt.ucc.asn.au/dropbear/dropbear.html](https://matt.ucc.asn.au/dropbear/dropbear.html)

#install dropbear
apt-get update
apt-get -y install dropbear
#change dropbear config
sed 's/^NO_START=1/NO_START=0/' /etc/default/dropbear > /tmp/db.$$
mv /tmp/db.$$ /etc/default/dropbear
#modify startup
update-rc.d -f dropbear remove
update-rc.d dropbear start 20 S . stop 20 0 6

After installation, you will see 22/tcp is listening by default.

**(d) Decrypting the signed file with passphrase**
The firmware.tar.gz has been encrypted, but this could be done with ease.
gpg --passphrase="$(cat MD5SUM).$(cat MODEL)" --decrypt firmware.tar.gz > dec_firmware.tar.gz


[![](http://3.bp.blogspot.com/-qxPdP6G9DUc/UOTVFrnLf4I/AAAAAAAAALA/3rmnjAuyyLk/s1600/gpg.jpg)](http://3.bp.blogspot.com/-qxPdP6G9DUc/UOTVFrnLf4I/AAAAAAAAALA/3rmnjAuyyLk/s1600/gpg.jpg)


The link below shows decryption method.
[http://svn.threnor.de/repos/N1T1/trunk/firmware/usr/lib/nas/firmware.sh](http://svn.threnor.de/repos/N1T1/trunk/firmware/usr/lib/nas/firmware.sh)


<blockquote>

>     
>     ### BEGIN_FIRMWARE_DECRYPTION
>     #
>     # $1: file
>     # $2: method
>     #
>     firmware_decryption() {
>       FILE="$1"
>       # Decrypt firmware
>       case "$2" in 
>         "METHOD_1")
>           gpg --passphrase="$(cat $(dirname $FILE)/MD5SUM).$NAS_MODEL" -d $FILE > ${FILE}.org
>           [ "$?" != 0 ] && return $?
>           mv -f ${FILE}.org $FILE
>           [ "$?" != 0 ] && return $?
>         ;;
>       esac
>       return 0
>     }
>     ### END_FIRMWARE_DECRYPTION
> 
> 
</blockquote>


Now you have full privilege, the system is your own.


