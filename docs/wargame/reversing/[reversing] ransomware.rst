============================================================================================================
[reversing] Ransomware
============================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c -> d;
        
        a [shape=box, color=lightblue, label="IDA"];
        b [shape=box, label="unpacking"];
        c [shape=box, label="dummy code -> nop"];
        d [shape=box, label="main()"];
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

    public start
    start           proc near
    
    var_AC          = byte ptr -0ACh
    
    pusha
    mov     esi, offset dword_ECD000
    lea     edi, [esi-0ACC000h]
    push    edi
    jmp     short loc_ECECFA
    .......
    push    edi
    call    ebp
    pop     eax
    popa        // 브레이킹 포인트
    lea     eax, [esp+2Ch+var_AC]
    
    loc_ECEE7A:                             ; CODE XREF: start+19Ej
    
    push    0
    cmp     esp, eax
    jnz     short loc_ECEE7A
    sub     esp, 0FFFFFF80h
    jmp     near ptr byte_44AC9B


- start 함수 부분에 pusha가 있는 것으로 보아 UPX로 패킹되어 있는 것으로 보이며, popa를 찾아 브레이킹 포인트를 걸어둡니다.
- step over를 진행하여 점프 명령까지 실행되면 UPX 언패킹된 상태의 코드를 확인할 수 있습니다.
- 언패킹된 상태의 코드를 "Take memory snapshot"을 통해 저장합니다. 

※ Edit -> Plugins -> Universal PE unpacker 로도 가능

|

dummy code -> nop
------------------------------------------------------------------------------------------------------------

언패킹시 output window에 아래와 같은 메시지가 출력되는데 더미 코드상에서 popad, pushad 등으로 스택에 저장했다 빼는 행위가 자주 반복되어 발생됩니다.

또한, 
.. code-block:: console

    4135E0: too many stack points have been declared
    4135E0: too many stack points have been declared
    4135E0: too many stack points have been declared

아래 더미 코드를 nop로 수정하여 해당 메시지가 발생되지 않도록 합니다.

.. code-block:: c

    pusha
    popa
    nop
    push eax
    pop eax
    push ebx
    pop ebx


6061905058535B -> 90909090909090

|

main
------------------------------------------------------------------------------------------------------------

IDA hexray decompiler로 확인해봅니다.

.. code-block:: c

    int __cdecl main(int argc, const char **argv, const char **envp)
    {
        unsigned int v3; // kr00_4@1
        FILE *v5; // [sp+1Ch] [bp-14h]@10
        unsigned __int32 v6; // [sp+20h] [bp-10h]@4
        int v7; // [sp+28h] [bp-8h]@1
        unsigned int i; // [sp+28h] [bp-8h]@7
        unsigned __int32 j; // [sp+28h] [bp-8h]@10
        FILE *File; // [sp+2Ch] [bp-4h]@1

        *(_BYTE *)(*(_DWORD *)(__readfsdword(24) + 48) + 2) = 68;
        CreateThread(0, 0, StartAddress, 0, 0, 0);
        printf("나는 나쁜놈이다!\n나는 매우 나쁘기 때문에 너의 파일을 암호화했다!\n너의 파일을 복구하고 싶다면 5천억 달러를 입금하고 받은 키값으로 파일을 복구해라!\n\n");
        printf("Key : ");
        sub_401000();
        scanf("%s", byte_44D370); // 입력값
        v3 = strlen(byte_44D370); // 입력값 길이
        sub_401000();
        v7 = 0;
        File = fopen("file", "rb"); // file 읽기
        sub_401000();
        if ( !File )
        {
            sub_401000();
            printf("\n\n\n파일을 찾을수 없다!\n");
            sub_401000();
            exit(0);
        }
        fseek(File, 0, 2);
        sub_401000();
        v6 = ftell(File); // 바이트 수 계산
        sub_401000();
        rewind(File);     // 파일 읽기/쓰기 위치를 처음 위치로 이동
        sub_401000();
        while ( !feof(File) )
        {
            sub_401000();
            byte_5415B8[v7] = fgetc(File);
            sub_401000();
            ++v7;
            sub_401000();
        }
        sub_401000();
        for ( i = 0; i < v6; ++i )
        {
            byte_5415B8[i] ^= byte_44D370[i % v3];
            sub_401000();
            byte_5415B8[i] = ~byte_5415B8[i];
            sub_401000();
        }
        fclose(File);
        sub_401000();
        v5 = fopen("file", "wb");
        sub_401000();
        sub_401000();
        for ( j = 0; j < v6; ++j )
        {
            fputc(byte_5415B8[j], v5);
            sub_401000();
        }
        printf("\n파일을 복구했다!\n나는 몹시 나쁘지만 약속은 지키는 사나이다!\n따라서 너가 나에게 돈을 줬고, 올바른 키값을 받았다면 파일은 정상화 되어 있을 것이다!\n하지만 만약 잘못된 키를 넣었다면 나는 아주아주 나쁘기 때문에 너의 파일은 또 망가질 것이다!");
        sub_401000();
        return getch();
    }

암호화된 file을 열어 입력한 키 값을 이용하여 복호화를 진행합니다.

readme.txt 파일 내용을 보면 Decrypt File이 EXE라고 되어 있으니, 
복호화될 파일의 헤더 부분이 4d 5a 90 00 으로 이루어질 것이라고 추측할 수 있습니다.

.. code-block:: text


    Decrypt File (EXE)


    By Pyutic



|

calculate
------------------------------------------------------------------------------------------------------------


디코딩 결과 키 값을 확인할 수 있습니다. 
    
.. code-block:: python

    # byte_5415B8[i] ^= byte_44D370[i % v3];
    # byte_5415B8[i] = ~byte_5415B8[i];

    output_file = ['4D', '5A', '90', '00', '03', '00', '00', '00', '04', '00', '00', '00', 'FF', 'FF', '00', '00']
    byte_5415b8 = ['DE', 'C0', '1B', '8C', '8C', '93', '9E', '86', '98', '97', '9A', '8C', '73', '6C', '9A', '8B']
    key = ''

    for l in range(16):
        output_file[l] = ord(output_file[l].decode('hex'))
        byte_5415b8[l] = ord(byte_5415b8[l].decode('hex'))
        byte_5415b8[l] = byte_5415b8[l] ^ 0xFF
        key += chr(byte_5415b8[l] ^ output_file[l])
        
    print key

