---
author: kevinkoo001@gmail.com
comments: true
date: 2012-09-09 16:55:07+00:00
layout: post
link: http://dandylife.net/blog/archives/41
slug: memory-forensics-with-volatility
title: Memory Forensics with Volatility
wordpress_id: 41
categories:
- Digital Forensic &amp; Investigation
tags:
- digital forensic
- memory forensic
- volatility
---

Volatility는 메모리 분석에 많이 사용되는 도구다. 많은 기능을 가지고 있으나 간단히 네트워크 연결, 레지스트리, 프로세스만 확인해 본 결과다. 이 밖에도 hive 파일, 메모리, 프로세스, LSA, hibernation 등을 덤프할 수 있으며 SSDT, 로드된 모듈, DLL 목록도 볼 수 있다. 사용된 대상 파일은 가상환경의 윈도우 XP SP3다.

Volatility is a popular tool to analyze memory. I checked network connections, registry hive structures and processes on the fly. Plus this tool helps you dump hive files, memory, process, LSA and hibernation as well as list SSDT, loaded DLLs and modules. The target file contains suspended Windows XP SP3 memory from virtual machine.

사용법은 아래를 참조하자.
D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe --info
Volatile Systems Volatility Framework 2.0

PLUGIN_COMMANDS
---------------
bioskbd      - Reads the keyboard buffer from Real Mode memory
connections  - Print list of open connections [Windows XP Only]
connscan     - Scan Physical memory for _TCPT_OBJECT objects (tcp connections)
crashinfo    - Dump crash-dump information
dlldump      - Dump DLLs from a process address space
dlllist      - Print list of loaded dlls for each process
driverscan   - Scan for driver objects _DRIVER_OBJECT
filescan     - Scan Physical memory for _FILE_OBJECT pool allocations
getsids      - Print the SIDs owning each process
handles      - Print list of open handles for each process
hashdump     - Dumps passwords hashes (LM/NTLM) from memory
hibinfo      - Dump hibernation file information
hivedump     - Prints out a hive
hivelist     - Print list of registry hives.
hivescan     - Scan Physical memory for _CMHIVE objects (registry hives)
imagecopy    - Copies a physical address space out as a raw DD image
imageinfo    - Identify information for the image
inspectcache - Inspect the contents of a cache
kdbgscan     - Search for and dump potential KDBG values
kpcrscan     - Search for and dump potential KPCR values
lsadump      - Dump (decrypted) LSA secrets from the registry
memdump      - Dump the addressable memory for a process
memmap       - Print the memory map
moddump      - Dump a kernel driver to an executable file sample
modscan      - Scan Physical memory for _LDR_DATA_TABLE_ENTRY objects
modules      - Print list of loaded modules
mutantscan   - Scan for mutant objects _KMUTANT
netscan      - Scan a Vista, 2008 or Windows 7 image for connections and sockets
patcher      - Patches memory based on page scans
printkey     - Print a registry key, and its subkeys and values
procexedump  - Dump a process to an executable file sample
procmemdump  - Dump a process to an executable memory sample
pslist       - print all running processes by following the EPROCESS lists
psscan       - Scan Physical memory for _EPROCESS pool allocations
pstree       - Print process list as a tree
sockets      - Print list of open sockets
sockscan     - Scan Physical memory for _ADDRESS_OBJECT objects (tcp sockets)
ssdt         - Display SSDT entries
strings      - Match physical offsets to virtual addresses (may take a while, VERY verbose)
testsuite    - Run unit test suit using the Cache
thrdscan     - Scan physical memory for _ETHREAD objects
userassist   - Print userassist registry keys and information
vaddump      - Dumps out the vad sections to a file
vadinfo      - Dump the VAD info
vadtree      - Walk the VAD tree and display in tree format
vadwalk      - Walk the VAD tree
volshell     - Shell in the memory image

PROFILES
--------
VistaSP0x86  - A Profile for Windows Vista SP0 x86
VistaSP1x86  - A Profile for Windows Vista SP1 x86
VistaSP2x86  - A Profile for Windows Vista SP2 x86
Win2K3SP0x86 - A Profile for Windows 2003 SP0 x86
Win2K3SP1x86 - A Profile for Windows 2003 SP1 x86
Win2K3SP2x86 - A Profile for Windows 2003 SP2 x86
Win2K8SP1x86 - A Profile for Windows 2008 SP1 x86
Win2K8SP2x86 - A Profile for Windows 2008 SP2 x86
Win7SP0x86   - A Profile for Windows 7 SP0 x86
Win7SP1x86   - A Profile for Windows 7 SP1 x86
WinXPSP2x86  - A Profile for Windows XP SP2
WinXPSP3x86  - A Profile for windows XP SP3

AS_CLASSES
----------
FileAddressSpace        - This is a direct file AS.
IA32PagedMemory         - Legacy x86 non PAE address space (to use specify --use_old_as)
IA32PagedMemoryPae      - Legacy x86 PAE address space (to use specify --use_old_as)
JKIA32PagedMemory       - Standard x86 32 bit non PAE address space.
JKIA32PagedMemoryPae    - Standard x86 32 bit PAE address space.
WindowsCrashDumpSpace32 - This AS supports windows Crash Dump format
WindowsHiberFileSpace32 - This is a hibernate address space for windows hibernation files.

SCANNER_CHECKS
--------------
CheckHiveSig           - Check for a registry hive signature
CheckPoolIndex         - Checks the pool index
CheckPoolSize          - Check pool block size
CheckPoolType          - Check the pool type
CheckProcess           - Check sanity of _EPROCESS
CheckSocketCreateTime  - Check that _ADDRESS_OBJECT.CreateTime makes sense
CheckThreads           - Check sanity of _ETHREAD
KPCRScannerCheck       - Checks the self referential pointers to find KPCRs
MultiStringFinderCheck - No docs
PoolTagCheck           - This scanner checks for the occurance of a pool tag

**1. Network connections**

PLUGIN마다 결과물이 다소 상이함에 주목하자.


**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.execonnections -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Offset(V)  Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ------
0x829a3008 192.168.80.131:3318       218.234.22.20:80            3700
0x829a3008 192.168.80.131:3415       1.227.196.103:80            3700
0x829a8300 192.168.80.131:3459       14.0.67.87:80               3700
0x829a8300 192.168.80.131:3455       14.0.67.87:80               3700
0x828bf008 192.168.80.131:3394       121.78.83.144:80            3700
0x828bf008 192.168.80.131:3398       121.78.83.144:80            3700
0x828ec470 192.168.80.131:3382       121.78.83.144:80            3700
0x828b1538 192.168.80.131:3352       74.125.128.113:80           3700
0x828764e0 192.168.80.131:3442       101.79.252.66:80            3700
0x828764e0 192.168.80.131:3402       101.79.252.66:80            3700
0x828ce890 192.168.80.131:3434       101.79.252.66:80            3700
0x82a7c688 192.168.80.131:3438       101.79.252.66:80            3700
0x828c9008 192.168.80.131:3371       66.220.147.93:443           3700
0x828c2ca8 192.168.80.131:3425       121.125.62.56:80            3700
0x82a06008 192.168.80.131:3383       218.234.22.20:80            3700
0x82a06008 192.168.80.131:3416       1.227.196.103:80            3700
0x828d4008 192.168.80.131:3448       14.0.67.87:80               3700
0x828d4008 192.168.80.131:3476       14.0.67.87:80               3700
0x8290f378 192.168.80.131:3472       14.0.67.87:80               3700
0x82a656a8 192.168.80.131:3395       121.78.83.144:80            3700
0x828bde70 192.168.80.131:3333       211.233.51.41:80            3700
0x829e3278 192.168.80.131:3309       202.43.63.139:80            3700
0x828a3820 192.168.80.131:3369       184.169.77.33:80            3700
0x828c6260 192.168.80.131:3439       101.79.252.66:80            3700
0x828c6260 192.168.80.131:3435       101.79.252.66:80            3700
0x829a5e70 192.168.80.131:3403       101.79.252.66:80            3700
0x829a8628 192.168.80.131:3443       101.79.252.66:80            3700
0x830f0008 192.168.80.131:3368       66.220.147.93:443           3700
0x829ac468 192.168.80.131:3450       211.218.153.52:80           3700
0x829ac468 192.168.80.131:3446       211.218.153.52:80           3700
0x82a86008 192.168.80.131:3294       211.218.152.62:80           3700
0x82878e70 192.168.80.131:3316       218.234.22.20:80            3700
0x82878e70 127.0.0.1:3297            127.0.0.1:5152              3700
0x82908d10 192.168.80.131:3417       1.227.196.103:80            3700
0x829a8a38 127.0.0.1:5152            127.0.0.1:3297               536
0x82848990 192.168.80.131:3469       14.0.67.87:80               3700
0x82848990 192.168.80.131:3453       14.0.67.87:80               3700
0x828c9830 192.168.80.131:3465       14.0.67.87:80               3700
0x828cfe70 192.168.80.131:3461       14.0.67.87:80               3700
0x828e09c0 192.168.80.131:3396       121.78.83.144:80            3700
0x828e09c0 192.168.80.131:3319       121.78.147.170:80           3700
0x82dbdb30 192.168.80.131:3347       121.78.147.170:80           3700
0x828af9d8 192.168.80.131:3326       211.233.51.41:80            3700
0x828af9d8 192.168.80.131:3346       211.233.51.41:80            3700
0x8297dae8 192.168.80.131:3306       74.125.128.157:80           3700
0x828c23e8 192.168.80.131:3366       184.169.77.33:80            3700
0x828f02b0 192.168.80.131:3440       101.79.252.66:80            3700
0x828f02b0 192.168.80.131:3436       101.79.252.66:80            3700
0x82beea80 192.168.80.131:3321       175.126.40.151:80           3700
0x82898d58 192.168.80.131:3447       211.218.153.52:80           3700
0x82866008 192.168.80.131:3419       211.233.33.12:80            3700
0x82866008 192.168.80.131:3466       14.0.67.87:80               3700
0x828f2510 192.168.80.131:3348       121.78.147.170:80           3700
0x828f2510 192.168.80.131:3397       121.78.83.144:80            3700
0x82a0b248 192.168.80.131:3423       211.233.51.41:80            3700
0x830dd6e8 192.168.80.131:3373       112.175.230.46:80           3700
0x82c5a990 192.168.80.131:3367       184.169.77.33:80            3700
0x828a2008 192.168.80.131:3441       101.79.252.66:80            3700
0x828a2008 192.168.80.131:3437       101.79.252.66:80            3700
0x82a09008 192.168.80.131:3401       101.79.252.66:80            3700
0x82a7d008 192.168.80.131:3393       101.79.252.66:80            3700
0x830db008 192.168.80.131:3370       66.220.147.93:443           3700




**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe connscan -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Offset     Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ------
0x02848990 192.168.80.131:3469       14.0.67.87:80               3700
0x02866008 192.168.80.131:3419       211.233.33.12:80            3700
0x028764e0 192.168.80.131:3442       101.79.252.66:80            3700
0x02878e70 192.168.80.131:3316       218.234.22.20:80            3700
0x02897e70 192.168.80.131:3028       121.78.147.170:80           3700
0x02898d58 192.168.80.131:3447       211.218.153.52:80           3700
0x028a2008 192.168.80.131:3441       101.79.252.66:80            3700
0x028a3820 192.168.80.131:3369       184.169.77.33:80            3700
0x028a63f8 192.168.80.131:3454       14.0.67.87:80               3700
0x028af258 1.0.0.0:25191             255.255.255.255:95          6060
0x028af9d8 192.168.80.131:3326       211.233.51.41:80            3700
0x028b1538 192.168.80.131:3352       74.125.128.113:80           3700
0x028bc3c8 0.0.166.130:1024          2.0.0.0:256                    0
0x028bde70 192.168.80.131:3333       211.233.51.41:80            3700
0x028bf008 192.168.80.131:3394       121.78.83.144:80            3700
0x028c23e8 192.168.80.131:3366       184.169.77.33:80            3700
0x028c2ca8 192.168.80.131:3425       121.125.62.56:80            3700
0x028c6260 192.168.80.131:3439       101.79.252.66:80            3700
0x028c9008 192.168.80.131:3371       66.220.147.93:443           3700
0x028c9830 192.168.80.131:3453       14.0.67.87:80               3700
0x028ce890 192.168.80.131:3402       101.79.252.66:80            3700
0x028cfe70 192.168.80.131:3465       14.0.67.87:80               3700
0x028d4008 192.168.80.131:3448       14.0.67.87:80               3700
0x028dc610 192.168.80.131:2743       121.78.83.152:80             620
0x028de6b8 192.168.80.131:3204       121.78.83.152:80            3700
0x028e09c0 192.168.80.131:3396       121.78.83.144:80            3700
0x028ec470 192.168.80.131:3398       121.78.83.144:80            3700
0x028f02b0 192.168.80.131:3440       101.79.252.66:80            3700
0x028f2510 192.168.80.131:3348       121.78.147.170:80           3700
0x028fcc98 192.168.80.131:3461       14.0.67.87:80               3700
0x02908d10 127.0.0.1:3297            127.0.0.1:5152              3700
0x0290a928 192.168.80.131:3462       14.0.67.87:80               3700
0x0290f378 192.168.80.131:3476       14.0.67.87:80               3700
0x02910350 192.168.80.131:3472       14.0.67.87:80               3700
0x02912c88 192.168.80.131:3436       101.79.252.66:80            3700
0x0297d860 192.168.80.131:3021       114.108.166.35:80           3700
0x0297dae8 192.168.80.131:3306       74.125.128.157:80           3700
0x029a08c8 192.168.80.131:3209       121.78.83.152:80            3700
0x029a3008 192.168.80.131:3318       218.234.22.20:80            3700
0x029a4e70 192.168.80.131:3464       14.0.67.87:80               3700
0x029a5e70 192.168.80.131:3435       101.79.252.66:80            3700
0x029a7e70 192.168.80.131:3207       121.78.83.152:80            3700
0x029a8300 192.168.80.131:3459       14.0.67.87:80               3700
0x029a8618 104.193.156.130:0         84.67.80.84:0                  0
0x029a8628 192.168.80.131:3403       101.79.252.66:80            3700
0x029a8a38 192.168.80.131:3417       1.227.196.103:80            3700
0x029ac468 192.168.80.131:3450       211.218.153.52:80           3700
0x029ae320 192.168.80.131:3471       14.0.67.87:80               3700
0x029ba4d0 192.168.80.131:3415       1.227.196.103:80            3700
0x029cc168 192.168.80.131:3443       101.79.252.66:80            3700
0x029cd380 192.168.80.131:3397       121.78.83.144:80            3700
0x029cda38 192.168.80.131:3468       14.0.67.87:80               3700
0x029e3278 192.168.80.131:3309       202.43.63.139:80            3700
0x029e3900 192.168.80.131:3455       14.0.67.87:80               3700
0x029ed8c0 192.168.80.131:3346       211.233.51.41:80            3700
0x029f1870 192.168.80.131:3038       121.78.147.170:80           3700
0x02a06008 192.168.80.131:3383       218.234.22.20:80            3700
0x02a06750 192.168.80.131:3216       211.169.247.231:80          3700
0x02a09008 192.168.80.131:3437       101.79.252.66:80            3700
0x02a0b248 192.168.80.131:3423       211.233.51.41:80            3700
0x02a0b538 192.168.80.131:3210       121.78.83.152:80            3700
0x02a0e548 192.168.80.131:3474       14.0.67.87:80               3700
0x02a1f8f8 192.168.80.131:3416       1.227.196.103:80            3700
0x02a656a0 0.0.0.0:21392             0.0.0.0:31054             2203101376
0x02a656a8 192.168.80.131:3395       121.78.83.144:80            3700
0x02a7c398 1.0.0.0:24832             255.255.255.255:0            956
0x02a7c688 192.168.80.131:3434       101.79.252.66:80            3700
0x02a7d008 192.168.80.131:3401       101.79.252.66:80            3700
0x02a86008 192.168.80.131:3446       211.218.153.52:80           3700
0x02beea80 192.168.80.131:3321       175.126.40.151:80           3700
0x02c45c08 192.168.80.131:3457       14.0.67.87:80               3700
0x02c49e70 192.168.80.131:3236       101.79.252.66:80            3700
0x02c5a990 192.168.80.131:3367       184.169.77.33:80            3700
0x02c5ae70 192.168.80.131:3039       121.78.147.170:80           3700
0x02d5ac68 192.168.80.131:3382       121.78.83.144:80            3700
0x02dbdb30 192.168.80.131:3319       121.78.147.170:80           3700
0x02e37d80 192.168.80.131:3007       202.43.63.144:80            3700
0x03094ab0 192.168.80.131:3040       121.78.147.170:80           3700
0x030d3690 192.168.80.131:3466       14.0.67.87:80               3700
0x030d7008 192.168.80.131:3438       101.79.252.66:80            3700
0x030da610 192.168.80.131:2664       115.165.181.27:8010         3256
0x030db008 192.168.80.131:3370       66.220.147.93:443           3700
0x030dd6e8 192.168.80.131:3373       112.175.230.46:80           3700
0x030dd8e8 1.0.0.0:0                 255.255.255.255:0            956
0x030e2e70 192.168.80.131:3393       101.79.252.66:80            3700
0x030f0008 192.168.80.131:3368       66.220.147.93:443           3700
0x030f4008 127.0.0.1:5152            127.0.0.1:3297               536
0x030f4b40 192.168.80.131:3238       101.79.252.66:80            3700
0x030fe008 192.168.80.131:3294       211.218.152.62:80           3700
0x030fe1e0 192.168.80.131:3452       14.0.67.87:80               3700
0x030ffa60 192.168.80.131:3347       121.78.147.170:80           3700




**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe sockets -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Offset(V)  PID    Port   Proto               Address        Create Time
---------- ------ ------ ------------------- -------------- --------------------------
0x829bc008   3700   3383      6 TCP            0.0.0.0            2012-09-08 13:58:53
0x829bc008   3700   3476      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x82a10c20   3700   3352      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x82a04008   3700   3321      6 TCP            0.0.0.0            2012-09-08 13:58:40
0x82a04008      4    138     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x82a075f8      4   1026      6 TCP            0.0.0.0            2012-09-08 04:51:16
0x82a01bd8   3700   3453      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x82a01bd8   3700   3294      6 TCP            0.0.0.0            2012-09-08 13:58:09
0x82a10008      4      0      1 ICMP           0.0.0.0            2012-09-08 04:51:05
0x828b8008   3700   3395      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x828b8008   3700   3333      6 TCP            0.0.0.0            2012-09-08 13:58:42
0x82a3ec98   1068    500     17 UDP            0.0.0.0            2012-09-08 04:51:19
0x828f8008   3700   3368      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x828f8008   3700   3461      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x82869008   3700   3306      6 TCP            0.0.0.0            2012-09-08 13:58:37
0x82869008   3700   3434      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x829a2cf0   3700   3465      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x82a8d008   3700   3403      6 TCP            0.0.0.0            2012-09-08 13:58:56
0x82c87c40   1484    123     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x82910c68   3700   3438      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x82910c68   3700   3469      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x829ae578      4    445      6 TCP            0.0.0.0            2012-09-08 04:50:45
0x828f7008   3700   3442      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x828f7008   3700   2929     17 UDP            127.0.0.1          2012-09-08 13:54:18
0x82cd9cc0   1352    135      6 TCP            0.0.0.0            2012-09-08 04:50:56
0x8286da18   3700   3318      6 TCP            0.0.0.0            2012-09-08 13:58:39
0x8286da18   3700   3415      6 TCP            0.0.0.0            2012-09-08 13:58:58
0x82910e98   3700   3446      6 TCP            0.0.0.0            2012-09-08 13:59:10
0x828f1e98   3700   3419      6 TCP            0.0.0.0            2012-09-08 13:59:00
0x828f1e98   3700   3450      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x829ddc98   3700   3423      6 TCP            0.0.0.0            2012-09-08 13:59:02
0x829ddc98   3700   3326      6 TCP            0.0.0.0            2012-09-08 13:58:40
0x830e2c30   3700   3396      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x828bd7e8   3700   3369      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x828eb008   3700   3466      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x828eb008   3700   3435      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x829a58c0   3700   3373      6 TCP            0.0.0.0            2012-09-08 13:58:50
0x829a4390   3700   3346      6 TCP            0.0.0.0            2012-09-08 13:58:45
0x829a4390   1484    123     17 UDP            127.0.0.1          2012-09-08 10:26:29
0x82d1c920   1068      0    255 Reserved       0.0.0.0            2012-09-08 04:51:19
0x82d48a20   3700   3439      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x830f7a00   3700   3443      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x8298dc20   3700   3447      6 TCP            0.0.0.0            2012-09-08 13:59:10
0x8298dc20   3700   3319      6 TCP            0.0.0.0            2012-09-08 13:58:40
0x8299d3e0   1720   1900     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x82d8cdf0   3700   3416      6 TCP            0.0.0.0            2012-09-08 13:58:58
0x82c60c20   1240   3389      6 TCP            0.0.0.0            2012-09-08 04:51:48
0x82c60c20   1412  33904      6 TCP            0.0.0.0            2012-09-08 04:51:31
0x82885218   3700   3393      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x82885218   3700   3455      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x82a1fe08      4    139      6 TCP            192.168.80.131     2012-09-08 10:26:29
0x828b8578   3700   3459      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x828b8578   3700   3366      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x828eaa98   3700   3397      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x8298a8b8   3700   3370      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x8298a8b8   3700   3401      6 TCP            0.0.0.0            2012-09-08 13:58:56
0x82a6c948      4      0     41 IPv6           0.0.0.0            2012-09-08 04:51:05
0x82c94c78   1420   1030      6 TCP            127.0.0.1          2012-09-08 04:51:44
0x82c94c78   3700   3436      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x82874c20   3700   3440      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x82874c20   3700   3347      6 TCP            0.0.0.0            2012-09-08 13:58:45
0x829b7008   3700   3382      6 TCP            0.0.0.0            2012-09-08 13:58:53
0x829b7008   3700   3316      6 TCP            0.0.0.0            2012-09-08 13:58:39
0x828a24f8   3700   3417      6 TCP            0.0.0.0            2012-09-08 13:58:58
0x828a24f8   3700   3448      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x828e9c98   1688  49619     17 UDP            0.0.0.0            2012-09-08 13:59:10
0x830ed2b0      4    137     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x82a24378   1720   1900     17 UDP            127.0.0.1          2012-09-08 10:26:29
0x828e67b0   3700   3297      6 TCP            0.0.0.0            2012-09-08 13:58:36
0x828e67b0   3700   3394      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x8299d008   3700   3425      6 TCP            0.0.0.0            2012-09-08 13:59:03
0x829e0008   1068   4500     17 UDP            0.0.0.0            2012-09-08 04:51:19
0x82896768   3700   3398      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x82896768   3700   3367      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x828c58c0   3700   3371      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x828c58c0   3700   3402      6 TCP            0.0.0.0            2012-09-08 13:58:56
0x828f04b0      4    445     17 UDP            0.0.0.0            2012-09-08 04:50:45
0x82e358d8    536   5152      6 TCP            127.0.0.1          2012-09-08 04:51:19
0x828848c0   3700   3437      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x828848c0   3700   3309      6 TCP            0.0.0.0            2012-09-08 13:58:38
0x829a8808   3700   3441      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x829a8808   3700   3348      6 TCP            0.0.0.0            2012-09-08 13:58:45
0x82a77c98      4      0     41 IPv6           192.168.80.131     2012-09-08 10:26:29
0x82d92ad8   3700   3472      6 TCP            0.0.0.0            2012-09-08 13:59:11




**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe sockscan -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Offset     PID    Port   Proto               Address        Create Time
---------- ------ ------ ------------------- -------------- --------------------------
0x02869008   3700   3306      6 TCP            0.0.0.0            2012-09-08 13:58:37
0x0286da18   3700   3318      6 TCP            0.0.0.0            2012-09-08 13:58:39
0x0286ec98   3700   3464      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02874c20   3700   3440      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x028848c0   3700   3437      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x02885218   3700   3393      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x02896768   3700   3398      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x028a24f8   3700   3417      6 TCP            0.0.0.0            2012-09-08 13:58:58
0x028a58f8   3700   3471      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x028b8008   3700   3395      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x028b8578   3700   3459      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x028bd7e8   3700   3369      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x028c4398   3700   3452      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x028c58c0   3700   3371      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x028cae98   3700   3457      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x028d42f8   3700   3347      6 TCP            0.0.0.0            2012-09-08 13:58:45
0x028e64b8   3700   3474      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x028e67b0   3700   3297      6 TCP            0.0.0.0            2012-09-08 13:58:36
0x028e9c98   3700   3448      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x028eaa98   3700   3366      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x028eb008   3700   3466      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x028f04b0   3700   3402      6 TCP            0.0.0.0            2012-09-08 13:58:56
0x028f1e98   3700   3419      6 TCP            0.0.0.0            2012-09-08 13:59:00
0x028f7008   3700   3442      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x028f8008   3700   3368      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x02910c68   3700   3438      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x02910e98   3700   3415      6 TCP            0.0.0.0            2012-09-08 13:58:58
0x0298a8b8   3700   3370      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x0298dc20   3700   3447      6 TCP            0.0.0.0            2012-09-08 13:59:10
0x0299d008   3700   3394      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x0299d3e0   3700   3319      6 TCP            0.0.0.0            2012-09-08 13:58:40
0x029a2cf0   3700   3434      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x029a4390   3700   3346      6 TCP            0.0.0.0            2012-09-08 13:58:45
0x029a58c0   3700   3435      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x029a8808   3700   3441      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x029ae578   3700   3469      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x029b7008   3700   3382      6 TCP            0.0.0.0            2012-09-08 13:58:53
0x029bc008   3700   3383      6 TCP            0.0.0.0            2012-09-08 13:58:53
0x029cec28   3700   3316      6 TCP            0.0.0.0            2012-09-08 13:58:39
0x029ddc98   3700   3423      6 TCP            0.0.0.0            2012-09-08 13:59:02
0x029dde98   3700   3473      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x029e0008   3700   3425      6 TCP            0.0.0.0            2012-09-08 13:59:03
0x029e58e0   3700   3461      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a01bd8   3700   3453      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a04008   3700   3321      6 TCP            0.0.0.0            2012-09-08 13:58:40
0x02a061e8   3700   3367      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x02a06778   3700   3241      6 TCP            0.0.0.0            2012-09-08 13:56:56
0x02a075f8      4    138     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x02a099c0   3700   3467      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a0d6e0   3700   3463      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a0d8b8   3700   3454      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a10008   3700   3294      6 TCP            0.0.0.0            2012-09-08 13:58:09
0x02a10c20   3700   3476      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a1e8d0   3700   3468      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a1fe08   3700   3455      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a24378   1720   1900     17 UDP            127.0.0.1          2012-09-08 10:26:29
0x02a3ec98   3700   3333      6 TCP            0.0.0.0            2012-09-08 13:58:42
0x02a4ac38   3700   3397      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x02a4fc58   3700   3462      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a6c948   3700   3401      6 TCP            0.0.0.0            2012-09-08 13:58:56
0x02a77c98   3700   3348      6 TCP            0.0.0.0            2012-09-08 13:58:45
0x02a7cb78   3700   3475      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02a85e98   3700   3203      6 TCP            0.0.0.0            2012-09-08 13:56:37
0x02a87008   3700   3309      6 TCP            0.0.0.0            2012-09-08 13:58:38
0x02a8c548   1688  57758     17 UDP            0.0.0.0            2012-09-08 13:56:35
0x02a8d008   3700   3465      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02c13d18      4    139      6 TCP            192.168.80.131     2012-09-08 10:26:29
0x02c60c20   1240   3389      6 TCP            0.0.0.0            2012-09-08 04:51:48
0x02c87c40   3700   3403      6 TCP            0.0.0.0            2012-09-08 13:58:56
0x02c94c78   1420   1030      6 TCP            127.0.0.1          2012-09-08 04:51:44
0x02cd9cc0   3700   2929     17 UDP            127.0.0.1          2012-09-08 13:54:18
0x02d1c920   1484    123     17 UDP            127.0.0.1          2012-09-08 10:26:29
0x02d48a20   1068      0    255 Reserved       0.0.0.0            2012-09-08 04:51:19
0x02d48e98   1068   4500     17 UDP            0.0.0.0            2012-09-08 04:51:19
0x02d4ace8   1068    500     17 UDP            0.0.0.0            2012-09-08 04:51:19
0x02d678b0      4      0      1 ICMP           0.0.0.0            2012-09-08 04:51:05
0x02d67b08      4      0     41 IPv6           0.0.0.0            2012-09-08 04:51:05
0x02d8cdf0   1720   1900     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x02d92ad8      4      0     41 IPv6           192.168.80.131     2012-09-08 10:26:29
0x02d9de98   3700   3472      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x02da1e98      4   1026      6 TCP            0.0.0.0            2012-09-08 04:51:16
0x02da3c28   3700   3352      6 TCP            0.0.0.0            2012-09-08 13:58:46
0x02dc6970   1352    135      6 TCP            0.0.0.0            2012-09-08 04:50:56
0x02e358d8      4    445     17 UDP            0.0.0.0            2012-09-08 04:50:45
0x02e618d8      4    445      6 TCP            0.0.0.0            2012-09-08 04:50:45
0x02f7b568   1412  33904      6 TCP            0.0.0.0            2012-09-08 04:51:31
0x02fd5808   1484    123     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x030a9b10    536   5152      6 TCP            127.0.0.1          2012-09-08 04:51:19
0x030d3008   3700   3373      6 TCP            0.0.0.0            2012-09-08 13:58:50
0x030d38b8   3700   3470      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x030d8008   3700   3446      6 TCP            0.0.0.0            2012-09-08 13:59:10
0x030dbc00   3700   3450      6 TCP            0.0.0.0            2012-09-08 13:59:11
0x030e2c30   3700   3396      6 TCP            0.0.0.0            2012-09-08 13:58:55
0x030ed2b0   1688  49619     17 UDP            0.0.0.0            2012-09-08 13:59:10
0x030efc20   3700   3439      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x030f1a60   3700   3326      6 TCP            0.0.0.0            2012-09-08 13:58:40
0x030f1cd8   3700   3416      6 TCP            0.0.0.0            2012-09-08 13:58:58
0x030f7a00   3700   3443      6 TCP            0.0.0.0            2012-09-08 13:59:09
0x030f8670      4    137     17 UDP            192.168.80.131     2012-09-08 10:26:29
0x030fee98   3700   3436      6 TCP            0.0.0.0            2012-09-08 13:59:09











**2. Processes**




** **




다음은 실제 가상환경에서 suspend하기 전 메모리 구조다. _EPROCESS 구조체를 따라 정확히 당시 환경을 추출해 주고 있다. psscan, pstree, pslist의 차이점을 비교해 보자.





Process PID CPU Private Bytes Working Set Description Company Name
System Idle Process 0 4.35 0 K 28 K
System 4 2.90 0 K 296 K
Interrupts n/a 7.25 0 K 0 K Hardware Interrupts and DPCs
smss.exe 448 172 K 432 K Windows NT Session Manager Microsoft Corporation
csrss.exe 988 1.45 3,092 K 5,920 K Client Server Runtime Process Microsoft Corporation
winlogon.exe 1012 8,360 K 4,428 K Windows NT Logon Application Microsoft Corporation
services.exe 1056 2,120 K 5,404 K Services and Controller app Microsoft Corporation
vmacthlp.exe 1228 736 K 2,680 K VMware Activation Helper VMware, Inc.
svchost.exe 1240 3,000 K 5,532 K Generic Host Process for Win32 Services Microsoft Corporation
wmiprvse.exe 5532 3,044 K 5,080 K WMI Microsoft Corporation
svchost.exe 1352 1,948 K 4,516 K Generic Host Process for Win32 Services Microsoft Corporation
svchost.exe 1484 1.45 13,584 K 22,240 K Generic Host Process for Win32 Services Microsoft Corporation
wuauclt.exe 3772 2,332 K 4,012 K Windows Update Microsoft Corporation
svchost.exe 1688 1,632 K 3,896 K Generic Host Process for Win32 Services Microsoft Corporation
svchost.exe 1720 2,944 K 5,508 K Generic Host Process for Win32 Services Microsoft Corporation
spoolsv.exe 1956 4,048 K 6,212 K Spooler SubSystem App Microsoft Corporation
WVSScheduler.exe 456 1,260 K 4,032 K Acunetix WVS Scheduler Acunetix Ltd.
jqs.exe 536 2,640 K 2,164 K Java(TM) Quick Starter Service Sun Microsystems, Inc.
npkcmsvc.exe 572 944 K 2,944 K nProtect KeyCrypt Manager Service INCA Internet Co., Ltd.
vmtoolsd.exe 888 6,508 K 8,320 K VMware Tools Core Service VMware, Inc.
WinCloud.exe 1412 12,988 K 9,780 K WinCloud Service Clunet
VMUpgradeHelper.exe 1644 1,176 K 4,240 K VMware virtual hardware upgrade helper application VMware, Inc.
alg.exe 1420 1,308 K 3,788 K Application Layer Gateway Service Microsoft Corporation
lsass.exe 1068 1.45 3,980 K 2,280 K LSA Shell (Export Version) Microsoft Corporation
explorer.exe 808 15,964 K 23,352 K Windows Explorer Microsoft Corporation
VMwareTray.exe 1132 2,304 K 5,132 K VMware Tools tray application VMware, Inc.
VMwareUser.exe 1140 2,872 K 6,788 K VMware Tools Service VMware, Inc.
ctfmon.exe 1072 1,024 K 3,456 K CTF Loader Microsoft Corporation
cmd.exe 6072 2,132 K 972 K Windows Command Processor Microsoft Corporation
conime.exe 4844 1,080 K 3,348 K Console IME Microsoft Corporation
iexplore.exe 3700 4.35 91,368 K 81,208 K Internet Explorer Microsoft Corporation
Tcpview.exe 3012 62.32 5,844 K 8,800 K TCP/UDP endpoint viewer Sysinternals - www.sysinternals.com
notepad.exe 2616 2,000 K 3,972 K Notepad Microsoft Corporation
procexp.exe 1276 14.49 14,304 K 17,796 K Sysinternals Process Explorer Sysinternals - www.sysinternals.com





**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe pslist -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Offset(V)  Name                 PID    PPID   Thds   Hnds   Time
---------- -------------------- ------ ------ ------ ------ -------------------
0x831b9830 System                    4      0     51    837 1970-01-01 00:00:00
0x8309c220 smss.exe                448      4      3     21 2012-09-08 04:50:45
0x83083b00 csrss.exe               988    448     13    508 2012-09-08 04:50:48
0x83071ba0 winlogon.exe           1012    448     19    507 2012-09-08 04:50:50
0x82fbf368 services.exe           1056   1012     15    287 2012-09-08 04:50:51
0x82fcd3b0 lsass.exe              1068   1012     20    353 2012-09-08 04:50:51
0x82e479a0 vmacthlp.exe           1228   1056      1     25 2012-09-08 04:50:53
0x830a1248 svchost.exe            1240   1056     19    221 2012-09-08 04:50:54
0x82e3a928 svchost.exe            1352   1056     10    309 2012-09-08 04:50:55
0x82e34928 svchost.exe            1484   1056     57   1314 2012-09-08 04:50:56
0x82d9fda0 svchost.exe            1688   1056      8    100 2012-09-08 04:51:02
0x82e669a0 svchost.exe            1720   1056     14    208 2012-09-08 04:51:03
0x82da8910 spoolsv.exe            1956   1056     11    127 2012-09-08 04:51:05
0x82dffa08 WVSScheduler.ex         456   1056      4     79 2012-09-08 04:51:18
0x830aabe0 jqs.exe                 536   1056      5    151 2012-09-08 04:51:18
0x82dc8b28 npkcmsvc.exe            572   1056      3     43 2012-09-08 04:51:19
0x82d12958 explorer.exe            808    780     15    563 2012-09-08 04:51:20
0x82cfeda0 vmtoolsd.exe            888   1056      5    244 2012-09-08 04:51:23
0x82e0ea78 VMwareTray.exe         1132    808      1     62 2012-09-08 04:51:24
0x82ceaab8 VMwareUser.exe         1140    808      4    162 2012-09-08 04:51:24
0x82ce3da0 ctfmon.exe             1072    808      1     76 2012-09-08 04:51:24
0x82f7d360 WinCloud.exe           1412   1056      9    160 2012-09-08 04:51:27
0x82e58da0 VMUpgradeHelper        1644   1056      3     96 2012-09-08 04:51:30
0x82c65b88 alg.exe                1420   1056      6    106 2012-09-08 04:51:44
0x82f5b9e0 wuauclt.exe            3772   1484      3    114 2012-09-08 04:52:40
0x828937c8 D                         0 3311691329      0 ------ 1970-01-01 00:00:00




**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe psscan -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Offset     Name             PID    PPID   PDB        Time created             Time exited
---------- ---------------- ------ ------ ---------- ------------------------ ------------------------
0x028a6da0 procexp.exe        1276    808 0x0b8803e0 2012-09-08 13:55:06
0x028afb88 notepad.exe        3204    808 0x0b880400 2012-09-08 13:57:24
0x028c3da0 Tcpview.exe        3012    808 0x0b8803c0 2012-09-08 13:54:55
0x0298d7d8 notepad.exe        2616    808 0x0b8803a0 2012-09-08 13:55:00
0x02a05960 wmiprvse.exe       5532   1240 0x0b8802a0 2012-09-08 13:54:57
0x02a1dae8 cmd.exe            6072    808 0x0b8802c0 2012-09-08 13:53:28
0x02c65b88 alg.exe            1420   1056 0x0b880260 2012-09-08 04:51:44
0x02ce3da0 ctfmon.exe         1072    808 0x0b880360 2012-09-08 04:51:24
0x02ceaab8 VMwareUser.exe     1140    808 0x0b880340 2012-09-08 04:51:24
0x02cfeda0 vmtoolsd.exe        888   1056 0x0b880280 2012-09-08 04:51:23
0x02d12958 explorer.exe        808    780 0x0b880240 2012-09-08 04:51:20
0x02d9a8b0 iexplore.exe       3700    808 0x0b880380 2012-09-08 13:54:16
0x02d9fda0 svchost.exe        1688   1056 0x0b880160 2012-09-08 04:51:02
0x02da8910 spoolsv.exe        1956   1056 0x0b8801a0 2012-09-08 04:51:05
0x02dc8b28 npkcmsvc.exe        572   1056 0x0b880200 2012-09-08 04:51:19
0x02dffa08 WVSScheduler.ex     456   1056 0x0b8801c0 2012-09-08 04:51:18
0x02e0ea78 VMwareTray.exe     1132    808 0x0b880320 2012-09-08 04:51:24
0x02e34928 svchost.exe        1484   1056 0x0b880140 2012-09-08 04:50:56
0x02e3a928 svchost.exe        1352   1056 0x0b880120 2012-09-08 04:50:55
0x02e479a0 vmacthlp.exe       1228   1056 0x0b8800c0 2012-09-08 04:50:53
0x02e58da0 VMUpgradeHelper    1644   1056 0x0b880300 2012-09-08 04:51:30
0x02e669a0 svchost.exe        1720   1056 0x0b880180 2012-09-08 04:51:03
0x02f5b9e0 wuauclt.exe        3772   1484 0x0b880220 2012-09-08 04:52:40
0x02f7d360 WinCloud.exe       1412   1056 0x0b880100 2012-09-08 04:51:27
0x02fbf368 services.exe       1056   1012 0x0b880080 2012-09-08 04:50:51
0x02fcd3b0 lsass.exe          1068   1012 0x0b8800a0 2012-09-08 04:50:51
0x03071ba0 winlogon.exe       1012    448 0x0b880060 2012-09-08 04:50:50
0x03083b00 csrss.exe           988    448 0x0b880040 2012-09-08 04:50:48
0x0309c220 smss.exe            448      4 0x0b880020 2012-09-08 04:50:45
0x030a1248 svchost.exe        1240   1056 0x0b8800e0 2012-09-08 04:50:54
0x030aabe0 jqs.exe             536   1056 0x0b8801e0 2012-09-08 04:51:18
0x030f2020 conime.exe         4844   6072 0x0b8802e0 2012-09-08 13:53:28
0x031b9830 System                4      0 0x00b20000




**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe pstree -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Name                                        Pid    PPid   Thds   Hnds   Time
0x831B9830:System                               4      0     51    837 1970-01-01 00:00:00
. 0x8309C220:smss.exe                          448      4      3     21 2012-09-08 04:50:45
.. 0x83071BA0:winlogon.exe                    1012    448     19    507 2012-09-08 04:50:50
... 0x82FBF368:services.exe                   1056   1012     15    287 2012-09-08 04:50:51
.... 0x82E58DA0:VMUpgradeHelper               1644   1056      3     96 2012-09-08 04:51:30
.... 0x82C65B88:alg.exe                       1420   1056      6    106 2012-09-08 04:51:44
.... 0x82D9FDA0:svchost.exe                   1688   1056      8    100 2012-09-08 04:51:02
.... 0x830AABE0:jqs.exe                        536   1056      5    151 2012-09-08 04:51:18
.... 0x82F7D360:WinCloud.exe                  1412   1056      9    160 2012-09-08 04:51:27
.... 0x82DA8910:spoolsv.exe                   1956   1056     11    127 2012-09-08 04:51:05
.... 0x82DFFA08:WVSScheduler.ex                456   1056      4     79 2012-09-08 04:51:18
.... 0x82E669A0:svchost.exe                   1720   1056     14    208 2012-09-08 04:51:03
.... 0x82DC8B28:npkcmsvc.exe                   572   1056      3     43 2012-09-08 04:51:19
.... 0x82E3A928:svchost.exe                   1352   1056     10    309 2012-09-08 04:50:55
.... 0x82E34928:svchost.exe                   1484   1056     57   1314 2012-09-08 04:50:56
..... 0x82F5B9E0:wuauclt.exe                  3772   1484      3    114 2012-09-08 04:52:40
.... 0x82E479A0:vmacthlp.exe                  1228   1056      1     25 2012-09-08 04:50:53
.... 0x830A1248:svchost.exe                   1240   1056     19    221 2012-09-08 04:50:54
..... 0x82A05960:wmiprvse.exe                 5532   1240      6    134 2012-09-08 13:54:57
.... 0x82CFEDA0:vmtoolsd.exe                   888   1056      5    244 2012-09-08 04:51:23
... 0x82FCD3B0:lsass.exe                      1068   1012     20    353 2012-09-08 04:50:51
.. 0x83083B00:csrss.exe                        988    448     13    508 2012-09-08 04:50:48
0x82D12958:explorer.exe                       808    780     15    563 2012-09-08 04:51:20
. 0x82CE3DA0:ctfmon.exe                       1072    808      1     76 2012-09-08 04:51:24
. 0x828C3DA0:Tcpview.exe                      3012    808     59    223 2012-09-08 13:54:55
. 0x82A1DAE8:cmd.exe                          6072    808      1     33 2012-09-08 13:53:28
.. 0x830F2020:conime.exe                      4844   6072      1     44 2012-09-08 13:53:28
. 0x82D9A8B0:iexplore.exe                     3700    808     43   1452 2012-09-08 13:54:16
. 0x828AFB88:notepad.exe                      3204    808      1     58 2012-09-08 13:57:24
. 0x8298D7D8:notepad.exe                      2616    808      1     55 2012-09-08 13:55:00
. 0x82E0EA78:VMwareTray.exe                   1132    808      1     62 2012-09-08 04:51:24
. 0x82CEAAB8:VMwareUser.exe                   1140    808      4    162 2012-09-08 04:51:24
. 0x828A6DA0:procexp.exe                      1276    808      9    303 2012-09-08 13:55:06











**3. Registry Hive Files**







아래는 레지스트리를 추출한 모습이다.







**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe hivescan -f winxp.vmem**
Volatile Systems Volatility Framework 2.0
Offset          (hex)
57266184        0x0369d008
57293664        0x036a3b60
60426072        0x039a0758
72600416        0x0453cb60
73767776        0x04659b60
186559328       0x0b1eab60
190231424       0x0b56b380
244668248       0x0e955758
245518888       0x0ea25228
276196192       0x10766b60
276494432       0x107af860
294652768       0x11900b60
574401376       0x223cab60




**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe hivelist -f winxp.vmem --profile=WinXPSP3x86**
Volatile Systems Volatility Framework 2.0
Virtual     Physical    Name
0xe2075b60  0x11900b60  \Device\HarddiskVolume1\Documents and Settings\Mr.Koo\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
0xe2098b60  0x223cab60  \Device\HarddiskVolume1\Documents and Settings\Mr.Koo\NTUSER.DAT
0xe1f3d860  0x107af860  \Device\HarddiskVolume1\Documents and Settings\LocalService\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
0xe1ea9b60  0x10766b60  \Device\HarddiskVolume1\Documents and Settings\LocalService\NTUSER.DAT
0xe1c1a228  0x0ea25228  \Device\HarddiskVolume1\Documents and Settings\NetworkService\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
0xe1c18758  0x0e955758  \Device\HarddiskVolume1\Documents and Settings\NetworkService\NTUSER.DAT
0xe1350380  0x0b56b380  \Device\HarddiskVolume1\WINDOWS\system32\config\software
0xe155cb60  0x04659b60  \Device\HarddiskVolume1\WINDOWS\system32\config\default
**0xe1516b60**  0x0453cb60  \Device\HarddiskVolume1\WINDOWS\system32\config\SAM
0xe17b9b60  0x0b1eab60  \Device\HarddiskVolume1\WINDOWS\system32\config\SECURITY
0xe132f758  0x039a0758  [no name]
**0xe1035b60**  0x036a3b60  \Device\HarddiskVolume1\WINDOWS\system32\config\system
0xe102e008  0x0369d008  [no name]
0x80670f20  0x00670f20  [no name]








hivelist 플러그인으로 알아낸 SYSTEM과 SAM의 가상 주소를 이용해 다음과 같이 -y -s 옵션과 함께 hashdump 플러그인을 사용해 해시값을 덤프할 수 있다. 여기서 LM Hash가 aad3b435b51404eeaad3b435b51404ee인 것은 빈 값임을 눈여겨 보자. 그리고 hack$ 계정이 숨김 속성으로 만들어져 있는 것으로 보아 이미 exploit되었을 가능성이 농후한 시스템이다.







Using SYSTEM and SAM virtual address from hivelist plugin, you can dump hash value of windows account with hashdump plugin, the option -y and -s. Make sure that "aad3b435b51404eeaad3b435b51404ee" value in LM Hash means an empty password. And hack$ account, hidden by default, indicates that this system had been highly exploited.





**D:\Tools\Digital Forensic\Memory\volatility\volatility-2.0.standalone>volatility.exe hashdump -f winxp.vmem --profile=WinXPSP3x86 -y 0xe1035b60 -s 0xe1516b60**

Volatile Systems Volatility Framework 2.0
Administrator:500:de655fbca508376***426435aa74e538:0cf564e9a7e683abd2ce9599863d54ec:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
HelpAssistant:1000:c44d37139c2b1b07d9f2ee77b096b984:71304be289c2b882d427de0dbecbe0a6:::
SUPPORT_388945a0:1002:aad3b435b51404eeaad3b435b51404ee:3d9c73f1aff9767791d113faeb3b4266:::
Mr.Koo:1003:2f91e790e5***880b0d3662b97ebed58:bdf7dca4108487c225d8aff46ffc4016:::
VUSR_VMXP:1004:adceda4626f31c007ba43c4ffd702e49:adc88278f39997873959184be8950691:::
hack$:1012:b757bf5c0d87772faad3b435b51404ee:7ce21f17c0aee7fb9ceba532d0546ad6:::
