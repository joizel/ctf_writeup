=====================================================================
[suninatas] Military
=====================================================================

메모리 덤프 파일 정보 확인
=====================================================================

.. code-block:: console

    $ vol.py imageinfo -f MemoryDump(SuNiNaTaS)

    Volatility Foundation Volatility Framework 2.3.1
    Determining profile based on KDBG search...

              Suggested Profile(s) : Win7SP0x86, Win7SP1x86
                         AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                         AS Layer2 : FileAddressSpace (/home/joizel/download/MemoryDump(SuNiNaTaS))
                          PAE type : PAE
                               DTB : 0x185000L
                              KDBG : 0x82f6cc28
              Number of Processors : 1
         Image Type (Service Pack) : 1
                    KPCR for CPU 0 : 0x82f6dc00
                 KUSER_SHARED_DATA : 0xffdf0000
               Image date and time : 2016-05-24 09:47:40 UTC+0000
         Image local date and time : 2016-05-24 18:47:40 +0900


|

프로세스 정보 확인
=====================================================================

위에서 나온 프로파일이 윈도우이므로, pslist로 프로세스 정보에 대해 확인합니다.

.. code-block:: console

    $ vol.py pslist --profile=Win7SP0x86  -f MemoryDump\(SuNiNaTaS\) 

    Volatility Foundation Volatility Framework 2.3.1
    Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
    ---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
    [......중략......]
    0x8779b9a0 iexplore.exe           3948   1408     27      465      1      0 2016-05-24 09:34:28 UTC+0000                                 
    0x8776d638 iexplore.exe           1840   3948     29      710      1      0 2016-05-24 09:34:45 UTC+0000                                 
    0x878fb650 iexplore.exe           2432   3948     52      840      1      0 2016-05-24 09:35:30 UTC+0000                                 
    0x85d72bc0 cmd.exe                1256   1408      1       22      1      0 2016-05-24 09:41:57 UTC+0000                                 
    0x878a6a88 conhost.exe            2920    368      3       87      1      0 2016-05-24 09:41:57 UTC+0000                                 
    0x8775b498 notepad.exe            3728   1256      2       89      1      0 2016-05-24 09:45:46 UTC+0000     <-- suspicious process                        
    0x856a4a58 SearchProtocol         4048    832      6      289      1      0 2016-05-24 09:46:47 UTC+0000                                 
    0x8569dd40 SearchFilterHo         2696    832      4       80      0      0 2016-05-24 09:46:48 UTC+0000                                 
    0x85e28b18 DumpIt.exe             1260   1408      2       37      1      0 2016-05-24 09:47:38 UTC+0000                                 
    0x856fa610 conhost.exe            1980    368      3       87      1      0 2016-05-24 09:47:38 UTC+0000    

|

IP 주소 확인
=====================================================================

.. code-block:: console

    $ vol.py netscan --profile=Win7SP0x86  -f MemoryDump\(SuNiNaTaS\) 

|

명령줄 확인
=====================================================================

.. code-block:: console

    $ vol.py cmdscan --profile=Win7SP0x86  -f MemoryDump\(SuNiNaTaS\) 

|

파일 확인
=====================================================================

.. code-block:: console

    $ vol.py filescan --profile=Win7SP0x86  -f MemoryDump\(SuNiNaTaS\) |grep Secreet

|

파일 덤프
=====================================================================

.. code-block:: console

    $ vol.py --profile=Win7SP0x86  -f MemoryDump\(SuNiNaTaS\) dumpfiles -Q [mem address] -D [out_dir]

|
