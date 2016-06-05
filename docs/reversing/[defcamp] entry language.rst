============================================================================================================
[defcamp] Entry language
============================================================================================================

.. uml::
    
    @startuml

    start

    :File Type;

    :strace;

    :IDA Hexray;
    
    stop

    @enduml

|

File Type
============================================================================================================

파일 하나를 주네요. file로 어떤 형식인지 살펴봅니다.

.. code-block:: console

    $ file ./r100
    ./r100: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), 
    dynamically linked (uses shared libs), for GNU/Linux 2.6.24, 
    BuildID[sha1]=0x2448460f21e38ecca7809aef1b0bc7996881ec6a, stripped

|

strace
============================================================================================================

stripped가 된 elf파일이기에 strace로 바이너리를 추적해봅니다.

.. code-block:: console

    $ strace ./r100
    ptrace(PTRACE_TRACEME, 0, 0, 0)         = -1 EPERM (Operation not permitted)

gdb로 디버깅 할 때 루프가 발생하도록 ptrace명령을 박아두었네요.

|

IDA Hexray
============================================================================================================

그럼 ida의 Hexray decompiler로 분석을 해보도록 하겠습니다.

먼저 start 함수 부분을 찾아 rdi로 이동하는 주소를 찾습니다.

그 주소가 바로 main 입니다. F5로 pseudo 코드를 보도록 하겠습니다.

.. code-block:: c

    signed __int64 sub_4007E8()
    {
        signed __int64 result; // rax@3
        __int64 v1; // rcx@6
        char s; // [sp+0h][bp-110h]@1
        __int64 v3; // [sp+108h][bp-8h]@1

        v3 = *MK_FP(__FS__, 40LL);
        printf("Enter thepassword: ");
        
        if ( fgets(&s, 255, stdin) )
        {
            if ( sub_4006FD(&s, 255LL) )
            {
                puts("Incorrectpassword!");
                result = 1LL;
            }
            
            else
            {
                puts("Nice!");
                result = 0LL;
            }
        }
        else
        {
            result = 0LL;
        }
        v1 = *MK_FP(__FS__, 40LL) ^ v3;
        return result;
    }



Nice가 나와야 하기 때문에 아마도 sub_4006FD에서 False(0)이면 될 것으로 보이네요. sub_4006FD를 따라가 봅니다.

.. code-block:: c

    signed __int64 __fastcall sub_4006FD(__int64 a1)
    {
    signed int i; // [sp+14h] [bp-24h]@1
    char v3[8]; // [sp+18h] [bp-20h]@1
    char v4[8]; // [sp+20h] [bp-18h]@1
    char v5[8]; // [sp+28h] [bp-10h]@1

    *(_QWORD *)v3 = "Dufhbmf";
    *(_QWORD *)v4 = "pG`imos";
    *(_QWORD *)v5 = "ewUglpt";
    for ( i = 0; i <= 11; ++i )
    {
        if ( *(_BYTE *)(*(_QWORD *)&v3[8 * (i % 3)] + 2 * (i / 3)) - *(_BYTE *)(i + a1) != 1 )
            return 1LL;
    }
    return 0LL;
    }


입력값 a1이 False가 되려면 *(_BYTE *)(*(_QWORD *)&v3[8 * (i % 3)] + 2 * (i / 3)) - *(_BYTE *)(i + a1) 이 1이 되야합니다.

*(_BYTE *)(*(_QWORD *)&v3[8 * (i % 3)] + 2 * (i / 3))의 결과 값을 x라는 리스트로 칭했을 때, 
x = [D,p,e,f,`,U,b,m,l,f,s,t] 입니다.

한마디로 x[i] - a1[i] = 1 이어야 합니다.


정답: Code_Talkers
