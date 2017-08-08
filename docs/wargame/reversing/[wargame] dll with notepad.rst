============================================================================================================
[wargame] dll with notepad
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

IDA
============================================================================================================

notepad에 출제자가 제작한 blueh4g13.dll이 로드되는 문제입니다. 

문제에서 시스템 시간을 확인하라는 힌트가 주어졌습니다.

로드되는 dll 파일의 소스를 hexray decompiler로 확인해봅니다.



start
------------------------------------------------------------------------------------------------------------

.. code-block:: c

    int __cdecl start(LPVOID lpParameter)
    {
        wchar_t *v1; // eax@2
        WCHAR Filename; // [sp+4h] [bp-204h]@1

        if ( GetModuleFileNameW(0, &Filename, 0x200u) )
        {
            v1 = wcsrchr(&Filename, 0x5Cu);
            if ( v1 )
            {
                if ( !wcsicmp(v1 + 1, L"notepad.exe") )
                {
                    dword_100033A8 = (int)CreateThread(0, 0, (LPTHREAD_START_ROUTINE)StartAddress, lpParameter, 0, 0);
                    CloseHandle((HANDLE)dword_100033A8);
                }
            }
        }
        return 1;
    }


time64와 localtime64_s라는 함수가 보입니다. 

|

sub_100010C0
------------------------------------------------------------------------------------------------------------


.. code-block:: c

    unsigned int sub_100010C0()
    {
        char *v0; // eax@1
        unsigned int v1; // esi@1
        char v2; // cl@2
        int v3; // ebx@4
        int v4; // edi@4
        int v5; // ecx@5
        signed int v6; // ecx@5
        unsigned int v7; // esi@8
        unsigned int result; // eax@8
        int v9; // ebx@9
        int v10; // edi@9
        int v11; // ecx@10
        signed int v12; // ecx@10
        __time64_t Time; // [sp+10h] [bp-90h]@1
        struct tm Tm; // [sp+18h] [bp-88h]@1
        char v15[32]; // [sp+3Ch] [bp-64h]@1
        char v16[64]; // [sp+5Ch] [bp-44h]@8

        Time = time64(0); // **
        localtime64_s(&Tm, &Time); // **
        dword_10003388 = 0;
        dword_1000338C = 0;
        dword_10003390 = 0;
        dword_10003394 = 0;
        dword_10003398 = 0;
        dword_1000339C = 0;
        dword_100033A0 = 0;
        dword_100033A4 = 0;
        memset((void *)&String, 0, 0x40u);
        sub_10001320("oh! handsome guy!");
        v0 = v15;
        v1 = 0;
        do
        v2 = *v0++;
        while ( v2 );
        if ( v0 != &v15[1] )
        {
            v3 = Tm.tm_mon;
            v4 = Tm.tm_mday * Tm.tm_mon;
            do
            {
                v5 = v4 + v15[v1];
                v6 = 126
                * (((signed int)(((unsigned __int64)(0x7DF7DF7Di64 * v5) >> 0x20) - v5) >> 6)
                + ((unsigned int)(((unsigned __int64)(0x7DF7DF7Di64 * v5) >> 0x20) - v5) >> 0x1F))
                + v5;
                if ( v6 < 33 )
                    v6 += v3 + 33;
                *((_BYTE *)&dword_10003388 + v1++) = v6;
                v4 += v3;
            }
            while ( v1 < strlen(v15) );
        }
        sub_10001340("Air fares to NY don't come cheap.");
        v7 = 0;
        result = strlen(v16);
        if ( result )
        {
            v9 = Tm.tm_mday;
            v10 = Tm.tm_mday * Tm.tm_mon;
            do
            {
                v11 = v10 + v16[v7];
                v12 = 126
                * (((signed int)(((unsigned __int64)(0x7DF7DF7Di64 * v11) >> 0x20) - v11) >> 6)
                + ((unsigned int)(((unsigned __int64)(0x7DF7DF7Di64 * v11) >> 0x20) - v11) >> 0x1F))
                + v11;
                if ( v12 < 33 )
                    v12 += v9 + 33;
                *((_BYTE *)&String + v7++) = v12;
                v10 += v9;
                result = strlen(v16);
            }
            while ( v7 < result );
        }
        result return;
    }


|

Immunity Debugger
============================================================================================================

Immunity Debugger에 time64와 localtime64_s 함수를 브레이킹 포인트를 걸고 실행을 하면 다음 어셈블리 코드 부분에서 멈추게 됩니다. 

.. image:: ../_images/reversing-02_1.png
    :align: center

step over로 계속 진행하다보면 SetWindowText 부분에 특정 문자열이 나오는데 해당 값을 입력하면 정답이 출력됩니다.

.. image:: ../_images/reversing-02_2.png
    :align: center
