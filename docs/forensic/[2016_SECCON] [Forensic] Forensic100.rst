==============================================================
[2016_SECCON] [Forensic] Forensic 100
==============================================================

문제내용
==============================================================

컴퓨터를 사용하다가 컴퓨터가 느려지는 현상이 발견되어, 원인을 파악해 보니 특정 파일에서 지속적으로 인터넷을 연결하는 현상이 감지 되었다. 해당 사이트에 접근해보니 특정 문구가 존재하였다.
해당 사이트에 접근하여 특정 문구인 flag를 획득하시오.

문제 풀이
==============================================================

메모리 덤프 파일을 주고, 분석을 진행하라고 하니 일단 volatility로 해당 덤프파일의 OS 확인

.. code-block:: console

    $ python vol.py imageinfo -f  forensic_100.raw
    Volatility Foundation Volatility Framework 2.6
    INFO    : volatility.debug    : Determining profile based on KDBG search...
              Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
                         AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                         AS Layer2 : FileAddressSpace (/home/joizel/ctf_test/forensic_100.raw)
                          PAE type : PAE
                               DTB : 0x34c000L
                              KDBG : 0x80545ce0L
              Number of Processors : 1
         Image Type (Service Pack) : 3
                    KPCR for CPU 0 : 0xffdff000L
                 KUSER_SHARED_DATA : 0xffdf0000L
               Image date and time : 2016-12-06 05:28:47 UTC+0000
         Image local date and time : 2016-12-06 14:28:47 +0900
         
    $ python vol.py -f forensic_100.raw --profile=WinXPSP2x86 pstree
    Volatility Foundation Volatility Framework 2.6
    Name                                                  Pid   PPid   Thds   Hnds Time
    -------------------------------------------------- ------ ------ ------ ------ ----
     0x8231f698:explorer.exe                             1556   1520     15    466 2016-12-06 05:27:10 UTC+0000
    . 0x821f8438:vmtoolsd.exe                            1856   1556      3    129 2016-12-06 05:27:11 UTC+0000
    . 0x819b4380:tcpview.exe                             3308   1556      2     84 2016-12-06 05:28:42 UTC+0000
    . 0x82267900:rundll32.exe                            1712   1556      2    144 2016-12-06 05:27:16 UTC+0000
    . 0x8216a5e8:DumpIt.exe                              3740   1556      1     25 2016-12-06 05:28:46 UTC+0000
    . 0x82170da0:ctfmon.exe                              1872   1556      1     87 2016-12-06 05:27:11 UTC+0000
     0x823c8660:System                                      4      0     58    259 1970-01-01 00:00:00 UTC+0000
    . 0x81a18020:smss.exe                                 540      4      3     19 2016-12-06 05:27:04 UTC+0000
    .. 0x82173da0:winlogon.exe                            628    540     24    541 2016-12-06 05:27:07 UTC+0000
    ... 0x8216e670:services.exe                           672    628     15    286 2016-12-06 05:27:07 UTC+0000
    .... 0x81f46238:alg.exe                              2028    672      7    104 2016-12-06 05:27:16 UTC+0000
    .... 0x82312450:svchost.exe                          1036    672     87   1514 2016-12-06 05:27:08 UTC+0000
    ..... 0x81f2cb20:wuauclt.exe                         3164   1036      5    107 2016-12-06 05:28:15 UTC+0000
    ..... 0x82062b20:wuauclt.exe                          488   1036      7    132 2016-12-06 05:27:13 UTC+0000
    ..... 0x81e56228:wscntfy.exe                          720   1036      1     37 2016-12-06 05:27:18 UTC+0000
    .... 0x82154880:vmacthlp.exe                          836    672      1     25 2016-12-06 05:27:08 UTC+0000
    .... 0x82151ca8:svchost.exe                           936    672     10    272 2016-12-06 05:27:08 UTC+0000
    .... 0x81e4b4b0:vmtoolsd.exe                          312    672      9    265 2016-12-06 05:27:13 UTC+0000
    .... 0x81f92778:svchost.exe                          1088    672      7     83 2016-12-06 05:27:08 UTC+0000
    .... 0x81f00558:VGAuthService.e                       196    672      2     60 2016-12-06 05:27:13 UTC+0000
    .... 0x81e18da0:svchost.exe                           848    672     20    216 2016-12-06 05:27:08 UTC+0000
    ..... 0x81e89200:wmiprvse.exe                         596    848     12    255 2016-12-06 05:27:13 UTC+0000
    .... 0x81e41928:svchost.exe                          1320    672     12    183 2016-12-06 05:27:10 UTC+0000
    .... 0x81f0dbe0:spoolsv.exe                          1644    672     15    133 2016-12-06 05:27:10 UTC+0000
    .... 0x81f65da0:svchost.exe                          1776    672      2     23 2016-12-06 05:27:10 UTC+0000
    ..... 0x8225bda0:IEXPLORE.EXE                         380   1776     22    385 2016-12-06 05:27:19 UTC+0000
    ...... 0x8229f7e8:IEXPLORE.EXE                       1080    380     19    397 2016-12-06 05:27:21 UTC+0000
    .... 0x81e4f560:svchost.exe                          1704    672      5    107 2016-12-06 05:27:10 UTC+0000
    ... 0x81f8c9a0:lsass.exe                              684    628     26    374 2016-12-06 05:27:07 UTC+0000
    .. 0x81ef6da0:csrss.exe                               604    540     11    480 2016-12-06 05:27:07 UTC+0000
     0x81e886f0:GoogleUpdate.ex                           372   1984      7    138 2016-12-06 05:27:13 UTC+0000
     
svchost.exe 프로세스에 IEXPLORE.EXE가 자식 프로세스로 실행되고 있는 것으로 보여 svchost.exe를 덤프

.. code-block:: console

    $ python vol.py -f VictimMemory.img --profile=WinXPSP2x86 procdump --pid=1776 --dump-dir=dump_file/
    Volatility Foundation Volatility Framework 2.6
    Process(V) ImageBase  Name                 Result
    ---------- ---------- -------------------- ------
    0x81e4f560 0x01000000 svchost.exe          OK: executable.1704.exe

덤프된 파일에 strings로 IEXPLORE.EXE가 존재하는 지 확인해본 결과 다음과 같은 URL로 접속 명령 진행 확인

.. code-block:: console

    $ strings dump_file/executable.1776.exe |grep -i iexplore.exe
    C:\Program Files\Internet Explorer\iexplore.exe http://crattack.tistory.com/entry/Data-Science-import-pandas-as-pd

해당 url에는 플래그가 존재하지 않았고, connections를 통해 연결 중인 네트워크를 확인해 본 결과 다음 IP에 접속함을 확인

.. code-block:: console

    $ nslookup http://crattack.tistory.com
    Server:         192.168.220.2
    Address:        192.168.220.2#53
    Non-authoritative answer:
    Name:   http://crattack.tistory.com
    Address: 175.126.170.110
    Name:   http://crattack.tistory.com
    Address: 175.126.170.70


    $ python vol.py -f forensic_100.raw --profile=WinXPSP2x86 connections
    Volatility Foundation Volatility Framework 2.6
    Offset(V)  Local Address             Remote Address            Pid
    ---------- ------------------------- ------------------------- ---
    0x8213bbe8 192.168.88.131:1034       153.127.200.178:80        1080

해당 IP를 통해 uri를 입력하여 접속하면 플래그 획득 가능하나, 지금은 서버가 종료된 상태임

.. code-block:: console

    $ curl http://153.127.200.178/entry/Data-Science-import-pandas-as-pd
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL /entry/Data-Science-import-pandas-as-pd was not found on this server.</p>
    </body></html>

