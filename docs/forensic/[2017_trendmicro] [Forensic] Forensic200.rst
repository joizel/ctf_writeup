==============================================================
[2017_trendmicro] [Forensic] Forensic200
==============================================================

문제내용
==============================================================

We got memory image from victim pc. Please analyze malicious indicator.

문제 풀이
==============================================================

메모리 덤프 파일을 주고, 분석을 진행하라고 하니 일단 volatility로 해당 덤프파일의 OS 확인

.. code-block:: console

    $ python vol.py imageinfo -f VictimMemory.img
    Volatility Foundation Volatility Framework 2.6
    INFO : volatility.debug : Determining profile based on KDBG search...
    Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86
    AS Layer1 : IA32PagedMemoryPae (Kernel AS)
    AS Layer2 : FileAddressSpace (/home/joizel/ctf_test/VictimMemory.img)
    PAE type : PAE
    DTB : 0x185000L
    KDBG : 0x8333ec28L
    Number of Processors : 1
    Image Type (Service Pack) : 1
    KPCR for CPU 0 : 0x8333fc00L
    KUSER_SHARED_DATA : 0xffdf0000L
    Image date and time : 2017-04-11 02:35:28 UTC+0000
    Image local date and time : 2017-04-11 11:35:28 +0900

프로세스 정보 확인

.. code-block:: console

    $ python vol.py -f VictimMemory.img --profile=Win7SP1x86_23418 pstree
    Volatility Foundation Volatility Framework 2.6
    Name                                                  Pid   PPid   Thds   Hnds Time
    -------------------------------------------------- ------ ------ ------ ------ ----
     0x89d8a530:wininit.exe                               412    344      3     78 2017-04-11 02:27:45 UTC+0000
    . 0x88a0c030:lsass.exe                                516    412      7    547 2017-04-11 02:27:48 UTC+0000
    . 0x88a056d8:services.exe                             508    412      7    220 2017-04-11 02:27:47 UTC+0000
    .. 0x869fa6c0:VSSVC.exe                              2304    508     12    194 2017-04-11 02:33:08 UTC+0000
    .. 0x89d91030:svchost.exe                            1288    508     17    304 2017-04-11 02:28:00 UTC+0000
    .. 0x86d7b030:VGAuthService.                         1424    508      3     87 2017-04-11 02:28:03 UTC+0000
    .. 0x89d6b030:mscorsvw.exe                           3096    508      6     74 2017-04-11 02:30:34 UTC+0000
    .. 0x88bd3a98:msdtc.exe                              1420    508     14    150 2017-04-11 02:28:28 UTC+0000
    .. 0x88a4bcd8:vmacthlp.exe                            676    508      3     53 2017-04-11 02:27:52 UTC+0000
    .. 0x88a808a0:svchost.exe                             808    508     20    465 2017-04-11 02:27:53 UTC+0000
    ... 0x88aa7130:audiodg.exe                            952    808      4    122 2017-04-11 02:27:55 UTC+0000
    .. 0x869b6030:msiexec.exe                            3612    508      9    278 2017-04-11 02:34:25 UTC+0000
    .. 0x89c0fb78:svchost.exe                            1668    508      8     92 2017-04-11 02:28:12 UTC+0000
    .. 0x86986030:sppsvc.exe                             3264    508      4    146 2017-04-11 02:30:44 UTC+0000
    .. 0x89a3b8e0:SearchIndexer.                         2376    508     12    576 2017-04-11 02:29:03 UTC+0000
    .. 0x88a87518:svchost.exe                             844    508     18    419 2017-04-11 02:27:53 UTC+0000
    ... 0x88b91030:dwm.exe                                568    844      3     70 2017-04-11 02:28:22 UTC+0000
    .. 0x86dcf2d0:vmtoolsd.exe                           1484    508      8    289 2017-04-11 02:28:07 UTC+0000
    ... 0x89a73d40:cmd.exe                               3880   1484      0 ------ 2017-04-11 02:35:27 UTC+0000
    .... 0x869b8d40:ipconfig.exe                         3900   3880      0 ------ 2017-04-11 02:35:28 UTC+0000
    .. 0x89d0b030:spoolsv.exe                            1232    508     12    326 2017-04-11 02:27:59 UTC+0000
    .. 0x86400838:taskhost.exe                           1976    508      9    165 2017-04-11 02:28:18 UTC+0000
    .. 0x8697fa58:svchost.exe                            3300    508      9    299 2017-04-11 02:30:45 UTC+0000
    .. 0x89a131f8:WmiApSrv.exe                           3728    508      5    111 2017-04-11 02:31:41 UTC+0000
    .. 0x88add030:svchost.exe                            1116    508     16    391 2017-04-11 02:27:57 UTC+0000
    .. 0x88a5e528:svchost.exe                             720    508      7    284 2017-04-11 02:27:53 UTC+0000
    .. 0x88a8baf8:svchost.exe                             868    508     42   1017 2017-04-11 02:27:53 UTC+0000
    .. 0x88a47130:svchost.exe                             616    508     10    359 2017-04-11 02:27:51 UTC+0000
    ... 0x89b5b5b0:WmiPrvSE.exe                          2108    616     10    294 2017-04-11 02:28:37 UTC+0000
    ... 0x88be3300:WmiPrvSE.exe                           204    616     10    204 2017-04-11 02:28:31 UTC+0000
    .. 0x88ab6c88:svchost.exe                            1008    508     13    282 2017-04-11 02:27:56 UTC+0000
    .. 0x8697bd40:svchost.exe                            3324    508      5     66 2017-04-11 02:33:09 UTC+0000
    .. 0x8694bd40:svchost.exe                            3192    508      9    126 2017-04-11 02:30:40 UTC+0000
    . 0x88a0ba38:lsm.exe                                  524    412     10    143 2017-04-11 02:27:48 UTC+0000
     0x86d1d7e8:csrss.exe                                 352    344      9    470 2017-04-11 02:27:43 UTC+0000
    . 0x86784030:conhost.exe                             3888    352      0 ------ 2017-04-11 02:35:28 UTC+0000
     0x8594b7e0:System                                      4      0     91    490 2017-04-11 02:27:39 UTC+0000
    . 0x86dd0d40:smss.exe                                 268      4      2     29 2017-04-11 02:27:39 UTC+0000
     0x89d83478:csrss.exe                                 404    396     10    199 2017-04-11 02:27:45 UTC+0000
    . 0x86938030:conhost.exe                             1868    404      3    100 2017-04-11 02:32:03 UTC+0000
     0x89da3530:winlogon.exe                              444    396      3    114 2017-04-11 02:27:45 UTC+0000
     0x88bbaab8:explorer.exe                              940    356     31    865 2017-04-11 02:28:23 UTC+0000
    . 0x8691c030:cmd.exe                                 4080    940      1     20 2017-04-11 02:32:02 UTC+0000
    .. 0x88abfa78:svchost.exe                            3828   4080      1      7 2017-04-11 02:35:18 UTC+0000
    . 0x88bca030:vmtoolsd.exe                            2216    940      6    191 2017-04-11 02:28:51 UTC+0000
    
cmd.exe에 svchost.exe가 자식 프로세스로 실행되고 있는 부분이 정상적이지 않아 보이므로 cmdscan을 통해 확인
"1.tmp 0x0 1"이라는 인자를 통해 svchost.exe에 실행된 것을 확인. svchost.exe는 프로세스 덤프로 덤프를 진행하고 1.tmp는 파일 덤프로 진행

.. code-block:: console

    $ python vol.py -f VictimMemory.img --profile=Win7SP1x86_23418 procdump --pid=3828 --dump-dir=dump_file/
    Volatility Foundation Volatility Framework 2.6
    Process(V) ImageBase  Name                 Result
    ---------- ---------- -------------------- ------
    0x88abfa78 0x00ed0000 svchost.exe          OK: executable.3828.exe
    $ python vol.py -f VictimMemory.img --profile=Win7SP1x86_23418 dumpfiles --regex=1.tmp --pid=3828 --dump-dir=dump_file/
    Volatility Foundation Volatility Framework 2.6
    DataSectionObject 0x88bb47c0   3828   \Device\HarddiskVolume1\Users\Taro\AppData\Local\Temp\1.tmp
    SharedCacheMap 0x88bb47c0   3828   \Device\HarddiskVolume1\Users\Taro\AppData\Local\Temp\1.tmp
    
svchost.exe 프로세스에 1.tmp파일이 메모리 인젝션되어 동작되는 것으로 보임.
1.tmp파일에 c9 c3(leave ret) 앞에 바이너리를 추출해서 메모리 인젝션 진행.

.. code-block:: console

    $ xxd dump_file/file.3828.0x86784ce0.vacb |head -n 30
    00000000: 9090 9090 9090 9090 9090 9090 9090 9090 ................
    00000010: 5589 e583 ec60 c645 daa8 c645 dbff c645 U....`.E...E...E
    00000020: dc88 c645 ddd0 c645 deb2 c645 dff6 c645 ...E...E...E...E
    00000030: e0f8 c645 e1ea c645 e2ff c645 e3ff c645 ...E...E...E...E
    00000040: e4d2 c645 e5ff c645 e6ff c645 e7c2 c645 ...E...E...E...E
    00000050: e8dc c645 e9c2 c645 ead8 c645 ebff c645 ...E...E...E...E
    00000060: ecf6 c645 edff c645 eefa c645 efff c645 ...E...E...E...E
    00000070: bc55 c645 bd8b c645 beec c645 bf51 c645 .U.E...E...E.Q.E
    00000080: c0e8 c645 c100 c645 c200 c645 c300 c645 ...E...E...E...E
    00000090: c400 c645 c558 c645 c62d c645 c752 c645 ...E.X.E.-.E.R.E
    000000a0: c81f c645 c934 c645 ca01 c645 cb2d c645 ...E.4.E...E.-.E
    000000b0: cc52 c645 cd1f c645 ce34 c645 cf01 c645 .R.E...E.4.E...E
    000000c0: d0e8 c645 d100 c645 d200 c645 d300 c645 ...E...E...E...E
    000000d0: d400 c645 d590 c645 d690 c645 d7c9 c645 ...E...E...E...E
    000000e0: d8c3 c645 d9cc c645 a600 c645 a75b c645 ...E...E...E.[.E
    000000f0: a800 c645 a900 c645 aa00 c645 ab00 c645 ...E...E...E...E
    00000100: ac00 c645 ad00 c645 ae2b c645 af17 c645 ...E...E.+.E...E
    00000110: b000 c645 b119 c645 b23f c645 b300 c645 ...E...E.?.E...E
    00000120: b400 c645 b500 c645 b600 c645 b703 c645 ...E...E...E...E
    00000130: b800 c645 b913 c645 ba00 c645 bb05 c745 ...E...E...E...E
    00000140: fc16 0000 00c7 45f4 0000 0000 c745 f000 ......E......E..
    00000150: 0000 008b 45f0 83f8 1673 708d 55da 8b45 ....E....sp.U..E
    00000160: f001 d00f b600 0fb6 c089 45f8 8d55 a68b ..........E..U..
    00000170: 45f0 01d0 0fb6 000f b6c0 8945 f483 7df4 E..........E..}.
    00000180: 007e 0a83 45f8 0183 6df4 01eb f08b 45fc .~..E...m.....E.
    00000190: 83e8 010f b644 05bc 0fb6 c029 45f8 8b45 .....D.....)E..E
    000001a0: fc83 e801 0fb6 4405 bc0f b6c0 3145 f8d1 ......D.....1E..
    000001b0: 7df8 8b45 f889 c18d 55da 8b45 f001 d088 }..E....U..E....
    000001c0: 0883 6dfc 0183 45f0 01eb 8890 c9c3 0000 ..m...E.........
    000001d0: 0000 0000 0000 0000 0000 0000 0000 0000 ................
 

    $ xxd -p dump_file/file.3828.0x86784ce0.vacb |tr -d '\n'|awk -F'c9c3' '{print $1'}
    909090909090909090909090909090905589e583ec60c645daa8c645dbffc645dc88c645ddd0c645deb2c645dff6c645e0f8c645e1eac645e2ffc645e3ffc645e4d2c645e5ffc645e6ffc645e7c2c645e8dcc645e9c2c645ead8c645ebffc645ecf6c645edffc645eefac645efffc645bc55c645bd8bc645beecc645bf51c645c0e8c645c100c645c200c645c300c645c400c645c558c645c62dc645c752c645c81fc645c934c645ca01c645cb2dc645cc52c645cd1fc645ce34c645cf01c645d0e8c645d100c645d200c645d300c645d400c645d590c645d690c645d7c9c645d8c3c645d9ccc645a600c645a75bc645a800c645a900c645aa00c645ab00c645ac00c645ad00c645ae2bc645af17c645b000c645b119c645b23fc645b300c645b400c645b500c645b600c645b703c645b800c645b913c645ba00c645bb05c745fc16000000c745f400000000c745f0000000008b45f083f81673708d55da8b45f001d00fb6000fb6c08945f88d55a68b45f001d00fb6000fb6c08945f4837df4007e0a8345f801836df401ebf08b45fc83e8010fb64405bc0fb6c02945f88b45fc83e8010fb64405bc0fb6c03145f8d17df88b45f889c18d55da8b45f001d08808836dfc018345f001eb8890
    
1.tmp에서 추출한 바이너리를 unicorn CPU 에뮬레이터에 로딩해서 플래그 획득

.. code-block:: python

    from __future__ import print_function
    from unicorn import *
    from unicorn.x86_const import *
    from binascii import *
    # code to be emulated
    X86_CODE32 = unhexlify("909090909090909090909090909090905589e583ec60c645daa8c645dbffc645dc88c645ddd0c645deb2c645dff6c645e0f8c645e1eac645e2ffc645e3ffc645e4d2c645e5ffc645e6ffc645e7c2c645e8dcc645e9c2c645ead8c645ebffc645ecf6c645edffc645eefac645efffc645bc55c645bd8bc645beecc645bf51c645c0e8c645c100c645c200c645c300c645c400c645c558c645c62dc645c752c645c81fc645c934c645ca01c645cb2dc645cc52c645cd1fc645ce34c645cf01c645d0e8c645d100c645d200c645d300c645d400c645d590c645d690c645d7c9c645d8c3c645d9ccc645a600c645a75bc645a800c645a900c645aa00c645ab00c645ac00c645ad00c645ae2bc645af17c645b000c645b119c645b23fc645b300c645b400c645b500c645b600c645b703c645b800c645b913c645ba00c645bb05c745fc16000000c745f400000000c745f0000000008b45f083f81673708d55da8b45f001d00fb6000fb6c08945f88d55a68b45f001d00fb6000fb6c08945f4837df4007e0a8345f801836df401ebf08b45fc83e8010fb64405bc0fb6c02945f88b45fc83e8010fb64405bc0fb6c03145f8d17df88b45f889c18d55da8b45f001d08808836dfc018345f001eb8890")
    # memory address where emulation starts
    ADDRESS = 0x1000000
    print("Emulate i386 code")
    try:
    # Initialize emulator in X64bit mode
        mu = Uc(UC_ARCH_X86, UC_MODE_32)
        # map 2MB memory for this emulation
        mu.mem_map(ADDRESS, 2 * 1024 * 1024)
        # write machine code to be emulated to memory
        mu.mem_write(ADDRESS, X86_CODE32)
        # initialize machine registers
        mu.reg_write(UC_X86_REG_ESP, ADDRESS + 0x100000)
        # emulate code in infinite time & unlimited instructions
        mu.emu_start(ADDRESS, ADDRESS + len(X86_CODE32))
        # now print out some registers
        print("Emulation done. Below is the CPU context")
        r_esp = mu.reg_read(UC_X86_REG_ESP)
        r_ebp = mu.reg_read(UC_X86_REG_EBP)
        print(">>> ESP = 0x%x" %r_esp)
        print(">>> EBP = 0x%x" %r_ebp)
        bytes_to_read = 0x60
        buf = mu.mem_read(r_esp, bytes_to_read)
        flag = ''
        for i in buf:
            try:
                print(chr(i))
                flag += chr(i)
            except:
                pass
        print(flag)
    except UcError as e:
        print("ERROR: %s" % e)
