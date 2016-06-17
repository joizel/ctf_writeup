============================================================================================================
[reversing] Ransomware
============================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c;
        c -> d [label="0",arrowhead="normal"];
        c -> e [label="1",arrowhead="normal"];
        
        a [shape=box, color=lightblue, label="IDA"];
        b [shape=box, label="sub_4007E8()"];
        c [shape=diamond, label="sub_4006FD()"];
        d [shape=box, label="Nice!"];
        e [shape=box, label="Incorrectpassword!"];
        
        
    }

|

압축을 풀면 run.exe, file, ReadMe.txt 3개의 파일이 존재합니다. 

run.exe를 이용해서 file을 복호화하는 것으로 보입니다.

IDA
============================================================================================================

manual unpacking
------------------------------------------------------------------------------------------------------------

먼저 run.exe를 IDA로 확인해봅니다.

.. code-block:: c

    UPX1:00ECECE0                 public start
    UPX1:00ECECE0 start           proc near
    UPX1:00ECECE0
    UPX1:00ECECE0 var_AC          = byte ptr -0ACh
    UPX1:00ECECE0
    UPX1:00ECECE0                 pusha
    UPX1:00ECECE1                 mov     esi, offset dword_ECD000
    UPX1:00ECECE6                 lea     edi, [esi-0ACC000h]
    UPX1:00ECECEC                 push    edi
    UPX1:00ECECED                 jmp     short loc_ECECFA
    .......
    UPX1:00ECEE71                 push    edi
    UPX1:00ECEE72                 call    ebp
    UPX1:00ECEE74                 pop     eax
    UPX1:00ECEE75                 popa        // 브레이킹 포인트
    UPX1:00ECEE76                 lea     eax, [esp+2Ch+var_AC]
    UPX1:00ECEE7A
    UPX1:00ECEE7A loc_ECEE7A:                             ; CODE XREF: start+19Ej
    UPX1:00ECEE7A                 push    0
    UPX1:00ECEE7C                 cmp     esp, eax
    UPX1:00ECEE7E                 jnz     short loc_ECEE7A
    UPX1:00ECEE80                 sub     esp, 0FFFFFF80h
    UPX1:00ECEE83                 jmp     near ptr byte_44AC9B


- start 함수 부분에 pusha가 있는 것으로 보아 UPX로 패킹되어 있는 것으로 보이며, popa를 찾아 브레이킹 포인트를 걸어둡니다.
- step over를 진행하여 점프 명령까지 실행되면 UPX 언패킹된 상태의 코드를 확인할 수 있습니다.
- 언패킹된 상태의 코드를 "Take memory snapshot"을 통해 저장합니다. 

※ Edit -> Plugins -> Universal PE unpacker 로도 가능

언패킹시 output window에 아래와 같은 메시지가 출력되는데 해당 코드에 더미 코드가 많다는 걸 추측할 수 있습니다.

.. code-block:: console

    4135E0: too many stack points have been declared

    MAX_ITEM_LINES          = 60000000



|

__tmainCRTStartup
------------------------------------------------------------------------------------------------------------


.. code-block:: c

    signed int __tmainCRTStartup()
    {
        LONG v0; // esi@3
        LONG v1; // eax@4
        int v3; // eax@21
        signed int v4; // [sp+14h] [bp-1Ch]@3

        if ( !dword_ECAC48 )
            HeapSetInformation(0, HeapEnableTerminationOnCorruption, 0, 0);
        v0 = *(_DWORD *)(__readfsdword(24) + 4);
        v4 = 0;
        while ( 1 )
        {
            v1 = InterlockedCompareExchange(&Target, v0, 0);
            if ( !v1 )
                break;
            if ( v1 == v0 )
            {
                v4 = 1;
                break;
            }
            Sleep(0x3E8u);
        }
        if ( dword_ECAC38 == 1 )
        {
            amsg_exit(31);
        }
        else if ( dword_ECAC38 )
        {
            dword_44D034 = 1;
        }
        else
        {
            dword_ECAC38 = 1;
            if ( initterm_e(&unk_44C0E0, &unk_44C0EC) )
                return 255;
        }
        if ( dword_ECAC38 == 1 )
        {
            initterm(&unk_44C0D4, &unk_44C0DC);
            dword_ECAC38 = 2;
        }
        if ( !v4 )
            InterlockedExchange(&Target, 0);
        if ( dword_ECAC4C && _IsNonwritableInCurrentImage(&dword_ECAC4C) )
            dword_ECAC4C(0, 2, 0);
        _initenv = envp;
        v3 = main(argc, (const char **)argv, (const char **)envp);
        dword_44D030 = v3;
        if ( !dword_44D024 )
            exit(v3);
        if ( !dword_44D034 )
            cexit();
        return dword_44D030;
    }

|

main
------------------------------------------------------------------------------------------------------------

