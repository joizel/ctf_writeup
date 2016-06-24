============================================================================================================
[hack-me] easy reversing
============================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c -> d;
        
        a [shape=box, color=lightblue, label="IDA"];
        b [shape=box, label="main()"];
        c [shape=box, label="sub_140001000()"];
        d [shape=box, label="decrypt"];
    }

|


IDA
============================================================================================================

IDA 디컴파일러(F5)로 디컴파일 하다보면 가끔 스택 문제도 실패하는 경우가 있습니다.

- 'Decompilation failure: xxxxxx: positive sp value has been found"
- 오류가 발생한 주소 앞 (여기서는 0x401018)으로 가서 [Alt + K] 로 Chage SP value 를 실행합니다.
- 이 값을 Current SP value 와 맞춰주면 됩니다.


wmain
------------------------------------------------------------------------------------------------------------


입력값에 상관없이 같은 결과 값을 출력하는 것으로 보아 저장된 값을 인코딩하여 출력하는 것으로 보입니다.

.. code-block:: c

    int wmain()
    {
        unsigned int v0; // et0@1
        int v1; // eax@1
        int v2; // eax@1
        int v3; // eax@1
        int v4; // eax@1
        int v5; // eax@1
        int v6; // eax@1
        int v7; // eax@1
        int v8; // eax@1
        int v9; // eax@1
        int v10; // eax@1
        int v11; // eax@1
        int v12; // eax@1
        int v13; // eax@1
        int v14; // eax@1
        int v15; // eax@1
        int v16; // eax@5
        int v18; // [sp+0h] [bp-50h]@1
        void *v19; // [sp+20h] [bp-30h]@1
        void *v20; // [sp+24h] [bp-2Ch]@1
        int v21; // [sp+28h] [bp-28h]@1
        char v22; // [sp+2Ch] [bp-24h]@1
        int *v23; // [sp+40h] [bp-10h]@1
        int v24; // [sp+4Ch] [bp-4h]@1

        v23 = &v18;
        v24 = 0;
        sub_401BC0(&v22);
        LOBYTE(v24) = 1;
        v21 = 0;
        v0 = __readeflags();
        v19 = &loc_40139F;
        v20 = &loc_4015E5;
        __writeeflags(v0);
        v21 = sub_401070(&loc_40139F, &loc_4015E5);
        sub_4010F0(0);
        v1 = sub_4019F0(std::cout, "t---------------------------------------t");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v1, std::endl);
        v2 = sub_4019F0(std::cout, "|                                       |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v2, std::endl);
        v3 = sub_4019F0(std::cout, "|       H a c k m e                     |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v3, std::endl);
        v4 = sub_4019F0(std::cout, "|                                       |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v4, std::endl);
        v5 = sub_4019F0(std::cout, "|             http://hack-me.org        |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v5, std::endl);
        v6 = sub_4019F0(std::cout, "|                                       |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v6, std::endl);
        v7 = sub_4019F0(std::cout, "|            Reverse Engineering        |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v7, std::endl);
        v8 = sub_4019F0(std::cout, "|                                       |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v8, std::endl);
        v9 = sub_4019F0(std::cout, "|                  Basic traning        |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v9, std::endl);
        v10 = sub_4019F0(std::cout, "|    by Bluebird                        |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v10, std::endl);
        v11 = sub_4019F0(std::cout, "|                                       |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v11, std::endl);
        v12 = sub_4019F0(std::cout, "|                                       |");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v12, std::endl);
        v13 = sub_4019F0(std::cout, "t---------------------------------------t");
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v13, std::endl);
        v14 = std::basic_ostream<char,std::char_traits<char>>::operator<<(std::cout, std::endl);
        v15 = std::basic_ostream<char,std::char_traits<char>>::operator<<(v14, std::endl);
        std::basic_ostream<char,std::char_traits<char>>::operator<<(v15, std::endl);
        sub_4019F0(std::cout, "[+] Enter Password : ");
        sub_4012B0();
        if ( (unsigned __int8)sub_4011B0() || (unsigned __int8)sub_401200() || (unsigned __int8)sub_4016A0() )
            TerminateProcess((HANDLE)0xFFFFFFFF, 4u);
        byte_44735C = sub_401750();
        v16 = unknown_libname_3(&v22);
        if ( (unsigned __int8)sub_401810(v16) )
            sub_401880();  // success
        else
            TerminateProcess((HANDLE)0xFFFFFFFF, 1u);
        if ( !(unsigned __int8)sub_4010B0(v21, v19, v20) )
            TerminateProcess((HANDLE)0xFFFFFFFF, 8u);
        LOBYTE(v24) = 0;
        sub_402480(&v22);
        return 0;
    }


패스워드가 출력되는 부분 함수를 확인해봅니다.

|


sub_401880
--------------------------------------------------------------------------------------------------

패스워드 출력 함수 부분을 보면 byte_4465D8에 있는 내용을 출력하는 것을 확인할 수 있습니다.

.. code-block:: c

    int sub_401880()
    {
        int v0; // eax@1

        sub_4019F0(std::cout, "Password is : ");
        v0 = sub_4019F0(std::cout, byte_4465D8);
        return std::basic_ostream<char,std::char_traits<char>>::operator<<(v0, std::endl);
    }

.. code-block:: python 

    byte_4465D8 = '''
    E0 F7 EA C7 F1 EB C7 FD F9 EB 
    F1 EB EC C7 FD F6 FB EA E1 E8 
    EC F1 F7 F6 C7 F5 FD EC F0 F7 
    FC F3 E4 F9 D4 E2 F8 D4 EE EA 
    F8 E2 F8 FF D4 EE E5 E8 F9 F2 
    FB FF E2 E4 E5 D4 E6 EE FF E3 
    E4 EF A7 B0 AD 80 B6 AC 80 BA 
    BE AC B6 AC AB 80 BA B1 BC AD 
    A6 AF AB B6 B0 B1 80 B2 BA AB 
    B7 B0 BB B4 A3 BE 93 A5 BF 93 
    A9 AD BF A5 BF B8 93 A9 A2 AF 
    BE B5 BC B8 A5 A3 A2 93 A1 A9 
    B8 A4 A3 A8 00 00 00 00
    '''

|


sub_401750
------------------------------------------------------------------------------------------------------------

byte_4465D8 값을 계산하는 부분에 대한 sub_401750 함수를 확인해봅니다.


.. code-block:: c


    char sub_401750()
    {
        signed int v0; // ebp@1
        char *v1; // eax@2
        char result; // al@5
        signed int i; // edi@11
        char v4; // bl@12
        unsigned int v5; // edx@12

        v0 = 0;
        if ( byte_4465D8[0] )
        {
            v1 = byte_4465D8;
            do
            {
                ++v1;
                ++v0;
            }
            while ( *v1 );
        }
        if ( byte_44743E && byte_4474AD && byte_44751C && byte_44758B && byte_4475FA && byte_447669 )
        {
            for ( i = 0x5D; i < v0; ++i )
            {
                v4 = 0;
                v5 = 0;
                if ( strlen(byte_447360) )
                {
                    do
                    {
                        if ( !byte_447360[v5] )
                            break;
                        ++v4;
                        ++v5;
                    }
                    while ( v5 < strlen(byte_447360) );
                }
                byte_4465D8[i] ^= v4 + byte_447669;
            }
            result = 1;
        }
        else
        {
            result = 0;
        }
        return result;
    }

byte_4465D8[93]부터 있는 값을 v4 + byte_447669와 xor 계산을 통해 출력하고 있습니다.

XOR 키 값(v4 + byte_447669)을 알지 못하기 때문에 Brute Force를 통해 키값을 구했습니다.

.. code-block:: python

    byte_4465D8 = '''E0 F7 EA C7 F1 EB C7 FD F9 EB 
    F1 EB EC C7 FD F6 FB EA E1 E8 
    EC F1 F7 F6 C7 F5 FD EC F0 F7 
    FC F3 E4 F9 D4 E2 F8 D4 EE EA 
    F8 E2 F8 FF D4 EE E5 E8 F9 F2 
    FB FF E2 E4 E5 D4 E6 EE FF E3 
    E4 EF A7 B0 AD 80 B6 AC 80 BA 
    BE AC B6 AC AB 80 BA B1 BC AD 
    A6 AF AB B6 B0 B1 80 B2 BA AB 
    B7 B0 BB B4 A3 BE 93 A5 BF 93 
    A9 AD BF A5 BF B8 93 A9 A2 AF 
    BE B5 BC B8 A5 A3 A2 93 A1 A9 
    B8 A4 A3 A8 00 00 00 00'''

    for x in range(255):
        ans = ''
        for l in byte_4465D8.split(' ')[93:]:
            a = int(l,16) ^ x
            #print l
            if 31<a and a<128 and a!=36 and a!=92 and a!=59:
                ans += chr(a)

        print x
        print ans



