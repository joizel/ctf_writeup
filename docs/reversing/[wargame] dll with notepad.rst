============================================================================================================
[wargame] dll with notepad
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

hexray decompiler
============================================================================================================

notepad에 출제자가 제작한 blueh4g13.dll이 로드되는 문제입니다. 

문제에서 시스템 시간을 확인하라는 힌트가 주어졌습니다.

로드되는 dll 파일의 소스를 hexray decompiler로 확인해봅니다.

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

|

Breaking Point
============================================================================================================

해당 dll에 시간과 관련된 호출 함수에 브레이킹 포인트를 설정합니다.
