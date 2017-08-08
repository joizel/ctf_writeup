============================================================================================================
[hack-me] M_IX64
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

main
------------------------------------------------------------------------------------------------------------

.. code-block:: c

    // local variable allocation has failed, the output may be wrong!
    int __cdecl main(int argc, const char **argv, const char **envp)
    {
        char *v3; // rcx@2
        int v4; // eax@3
        int v5; // edx@3

        sub_1400011F0(*(_QWORD *)&argc, argv, envp);
        printf("Enter Password : ");
        gets(Buffer);
        if ( 4 * (3 * ((unsigned int)strlen(Buffer) + 1) - 3) != 252 ) // (unsigned int)strlen(Buffer) = 21
            goto LABEL_11;
        sub_140001000(Buffer); // Dest
        printf("%s\n", Dest);
        ㅇ = a6c6573706c1194;
        do
        {
            v4 = (unsigned __int8)v3[Dest - a6c6573706c1194]; // 6C65-7370-6C119-4083-4593-10744-5194-964E-6A53-4180-7D
            v5 = (unsigned __int8)*v3 - v4;
            if ( (unsigned __int8)*v3 != v4 )
                break;
            ++v3;
        }
        while ( v4 );
        if ( v5 )
    LABEL_11:
            printf("Nope, You are worng :)\n");
        else
            printf("Congratulations! :) Password is %s\n", Buffer);   // success
        return 0;
    }

Dest 값이 위의 a6c6573706c1194에 있는 값과 일치할 경우에 통과되는 것으로 보입니다.

Dest 값이 출력되는 sub_140001000 함수를 확인해봅니다.

|

sub_140001000
------------------------------------------------------------------------------------------------------------

.. code-block:: c

    int sub_140001000()
    {
        signed __int64 v0; // r10@1
        unsigned int v1; // er8@1
        signed __int64 v2; // r9@1
        __int64 v3; // rax@2
        int v4; // er8@2
        char v5; // al@3

        v0 = 0i64;
        v1 = 0xDEADBEAF;
        v2 = 0i64;
        do
        {
            ++v2;
            v3 = v1 % 0x1A;         // 0xD, 0xF
            v4 = __ROL4__(v1, 3);   // 0x6F56DF578
            v1 = v4 + 0xF00D;       // 0x6F56EE585
            *(_BYTE *)(v2 + 0x140003637i64) = *(_BYTE *)(v3 + 0x140003020i64); // 0x4e 
        }
        while ( v2 < 21 );
        do
        {
            v5 = *(_BYTE *)(v0 + 0x140003690i64); // 0x62
            v0 += 3i64;
            *(_BYTE *)(v0 + 0x140003635i64) ^= v5 - 0x30; // 0x140003638i64 ^ (v5 - 0x30) = 0x4e ^ 0x22 = 0x6C
            *(_BYTE *)(v0 + 0x140003636i64) ^= *(_BYTE *)(v0 + 0x14000368Ei64) - 0x30; // 0x140003639i64 ^ (0x140003691i64 - 0x30) = 0x4f ^ 0x32 = 7A
            *(_BYTE *)(v0 + 0x140003637i64) ^= *(_BYTE *)(v0 + 0x14000368Fi64) - 0x30; // 0x140003640i64 ^ (0x140003692i64 - 0x30) = 
        }
        while ( v0 < 21 );
        return sprintf(
                   Dest,
                   "%lX%lu-%lX%lu-%lX%lu-%lX%lu-%lX%lu-%lu%lX-%lX%lu-%lu%lX-%lX%lX-%lX%lu-%lX",
                   (unsigned __int8)byte_140003638,
                   (unsigned __int8)byte_140003639);
    }


첫번째 do while 문을 보면 0x140003637i64 에 차례대로 21개의 데이터를 저장하는 것을 볼 수 있습니다.

그리고 두번째 do while 문을 보면 위에서 저장한 데이터를 사용자가 입력한 값 0x140003690i64와 차례대로 계산하는 것을 볼 수 있습니다.

여기서 계산한 결과 값이 Dest값으로 해당 값이 저장된 값과 일치해야 정답이 출력됩니다.

위 코드를 python으로 변환하면 아래와 같습니다.

.. code-block:: python

    _input = 'REsdefghijklmnopqrstu'

             #%X%u-%X%u-%X%u-%X%u-%X%u- %u%X-%X%u-%u%X-%X%X-%X%u-%X
    result = "6C65-7370-6C119-4083-4593-10744-5194-964E-6A53-4180-7D"

    def rol_Dword(value, rotation):
        tail = value >> (32-rotation)
        head = (value - (tail << (32 - rotation))) << rotation
        return head + tail
    def ror_Dword(value, rotation):
        tail = value >> rotation
        head = (value - (tail << rotation)) << (32 - rotation)
        return head + tail

    v1 = 0xDEADBEAF
    li = []
    for i in range(21):
        v3 = v1 % 0x1a
        v4 = rol_Dword(v1,3)
        v1 = v4 + 0xF00D
        li.append(v3+0x41)

    #print li
    #print ','.join([hex(i) for i in li])

    for j in range(7):
        li[3*j] ^= int(_input[3*j].encode('hex'),16) - 0x30
        li[3*j+1] ^= int(_input[3*j+1].encode('hex'),16) -0x30
        li[3*j+2] ^= int(_input[3*j+2].encode('hex'),16) -0x30

    #print li

|

decrypt
------------------------------------------------------------------------------------------------------------

해당 코드를 디코딩하면 됩니다. 디코딩 함수는 아래와 같습니다.

.. code-block:: python

    cal_li = [0x4e,0x54,0x55,0x53,0x4e,0x54,0x55,0x4e,0x50,0x4f,0x42,0x57,0x49,0x4b,0x42,0x57,0x49,0x4b,0x52,0x41,0x59]
    resul2 = "6C4173466C774053455d6b44515e604E6A5341507D".decode('hex')

    ans = ''
    for x in range(21):
        n = resul2[x].encode('hex')
        n = int(n,16)
        #print n
        n = (n^cal_li[x]) + 0x30
        ans += chr(n)

    print ans
