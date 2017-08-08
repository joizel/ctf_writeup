============================================================================================================
[2016_hackover] [PWN] tiny_backdoor_v1
============================================================================================================

|

Dynamic Analysis
============================================================================================================

.. code-block:: console


    $ ./tiny_backdoor_v1

|

Static Analysis
============================================================================================================

먼저 IDA Hexray를 이용하여 해당 프로그램의 소스를 확인해보면 다음과 같습니다.

.. code-block:: c

    int __fastcall sub_4000F9(_BYTE *a1, __int64 a2, unsigned __int64 a3, unsigned __int64 a4)
    {
        unsigned __int64 v4; // r9@1
        __int64 v5; // r10@1
        unsigned __int64 v6; // rsi@1
        unsigned __int64 v7; // r8@1
        __int64 v8; // rcx@1

        v4 = a4;
        v5 = a2;
        v6 = a3;
        v7 = 0LL;
        v8 = 0LL;
        while ( v8 != v5 )
        {
            a1[v8] ^= *(_BYTE *)(v6 + v7);
            v8 = (unsigned int)(v8 + 1);
            a3 = (v7 + 1) % v4;
            v7 = (v7 + 1) % v4;
        }
        return ((int (__fastcall *)(_BYTE *, unsigned __int64, unsigned __int64, __int64, unsigned __int64, unsigned __int64))a1)(
        a1,
        v6,
        a3,
        v8,
        v7,
        v4);
    }
    
|

