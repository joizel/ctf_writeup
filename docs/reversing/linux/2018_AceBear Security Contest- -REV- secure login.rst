==============================================================
[2018_AceBear Security Contest] [REV] secure login
==============================================================


문제내용
==============================================================


Please run solution using ubuntu 16.04
Time of server is UTC+000
Service: nc securelogin.acebear.site 5001


문제 풀이
==============================================================

먼저 바이너리 파일을 리눅스에서 실행해보면 다음과 같다.

.. code-block:: sh

    $ ./secure_login
    **************************Welcome to secure login**************************
    *                                                                         *
    *************************Challenge Created By CNV**************************
    *   Team: AceBear                                                         *
    *   My blog: https://chung96vn.blogspot.com/                              *
    ***************************************************************************
    Current time: Tue Jan 30 20:42:24 2018

    Give me your name: 
    Welcome: 
    Gime me your password: 
    Generated password: 
    $ 

name과 password를 입력값으로 하여 일치여부를 확인하는 문제이다.

|

IDA hexray decompiler
---------------------------------------------------------------------

제공해준 파일의 코드 중 패스워드를 암호화 하는 부분의 코드는 다음과 같았다. 
패스워드 암호화 시 rand함수를 사용한다.

.. code-block:: c

    void *__cdecl sub_8048950(int a1)
    {
        __int16 v1; // ST24_2@5
        __int16 v2; // ST28_2@5
        __int16 v3; // ax@5
        signed int i; // [sp+0h] [bp-28h]@4
        int v6; // [sp+4h] [bp-24h]@4
        void *s; // [sp+8h] [bp-20h]@1
        char *nptr; // [sp+Ch] [bp-1Ch]@4
        char *v9; // [sp+10h] [bp-18h]@4

        s = malloc(0x40u);
        if ( !s )
        {
            puts("Can not malloc!");
            exit(0);
        }
        memset(s, 0, 0x40u);
        nptr = (char *)malloc(0x10u);
        v9 = (char *)malloc(0x10u);
        LOWORD(v6) = 0;
        for ( i = 0; i <= 15; ++i )
        {
            *(_DWORD *)nptr = *(_DWORD *)(4 * i + a1);
            *(_DWORD *)v9 = dword_804B0C0[i];
            v1 = strtoul(nptr, (char **)nptr + 2, 16);
            v2 = strtoul(v9, (char **)v9 + 2, 16);
            v3 = rand();
            v6 = (unsigned __int16)(v2 * ((v6 ^ v1 ^ v3) + 1) + (v6 ^ v1 ^ v3));
            sprintf((char *)s + 4 * i, "%04X", v6);
        }
        return s;
    }


rand함수가 있으므로 seed가 존재하는 지 찾아보니 실행 시 Current time을 뿌려주면서 seed도 결정하는 것으로 보인다.

.. code-block:: c

    seed = time(0);
    srand(seed);



radare2를 통한 디버깅
---------------------------------------------------------------------


.. code-block:: sh

    [0xf77d6a20]> aaa
    [x] Analyze all flags starting with sym. and entry0 (aa)
    [x] Analyze function calls (aac)
    [x] Analyze len bytes of instructions for references (aar)
    [x] Use -AA or aaaa to perform additional experimental analysis.
    [x] Constructing a function name for fcn.* and sym.func.* functions (aan)
    = attach 11008 11008
    11008
    [0xf77d6a20]> dcu main
    Continue until 0x08048bc4 using 1 bpsize
    hit breakpoint at: 8048bc4

기본 명령어

.. code-block:: sh
    
    V + p + p (disassembly)
    step into (s)
    step over (S)
    :px 200@[address]

for 문이 동작하는 부분을 집중적으로 디버깅하며 입력값의 변화를 확인하자.
패스워드는 입력값 "1234567890123456789012345678901234567890123456789012345678901234"을 입력하였다.

.. code-block:: asm

    .text:08048A74 loc_8048A74:
    .text:08048A74    cmp     [ebp+var_28], 0Fh
    .text:08048A78    jle     loc_80489CB


.. code-block:: asm

    .text:080489CB loc_80489CB:
    .text:080489CB    mov     eax, [ebp+var_28]     // i=0~15
    .text:080489CE    shl     eax, 2                // 4*i
    .text:080489D1    mov     edx, eax
    .text:080489D3    mov     eax, [ebp+arg_0]
    .text:080489D6    add     eax, edx              // a1[4*i]
    .text:080489D8    mov     edx, [eax]
    .text:080489DA    mov     eax, [ebp+nptr]
    .text:080489DD    mov     [eax], edx            // nptr=eax
    .text:080489DF    mov     eax, [ebp+var_28]     // i=0~15
    .text:080489E2    shl     eax, 2                // 4*i
    .text:080489E5    add     eax, 804B0C0h
    .text:080489EA    mov     edx, [eax]
    .text:080489EC    mov     eax, [ebp+var_18]
    .text:080489EF    mov     [eax], edx
    .text:080489F1    mov     eax, [ebp+nptr]
    .text:080489F4    add     eax, 8
    .text:080489F7    sub     esp, 4
    .text:080489FA    push    10h             ; base
    .text:080489FC    push    eax             ; endptr
    .text:080489FD    push    [ebp+nptr]      ; nptr
    .text:08048A00    call    _strtoul
    .text:08048A05    add     esp, 10h
    .text:08048A08    mov     [ebp+var_14], eax     // v1
    .text:08048A0B    mov     eax, [ebp+var_18]
    .text:08048A0E    add     eax, 8
    .text:08048A11    sub     esp, 4
    .text:08048A14    push    10h             ; base
    .text:08048A16    push    eax             ; endptr
    .text:08048A17    push    [ebp+var_18]    ; nptr
    .text:08048A1A    call    _strtoul
    .text:08048A1F    add     esp, 10h
    .text:08048A22    mov     [ebp+var_10], eax     // v2
    .text:08048A25    call    _rand
    .text:08048A2A    movzx   eax, ax               // v3=eax
    .text:08048A2D    xor     eax, [ebp+var_14]     // v3^v1
    .text:08048A30    xor     eax, [ebp+var_24]     // v3^v1^v6
    .text:08048A33    mov     [ebp+var_C], eax      
    .text:08048A36    mov     eax, [ebp+var_C]
    .text:08048A39    add     eax, 1                // v3^v1^v6 + 1
    .text:08048A3C    imul    eax, [ebp+var_10]     // (v3^v1^v6 + 1) * v2
    .text:08048A40    mov     edx, eax
    .text:08048A42    mov     eax, [ebp+var_C]      // (v3^v1^v6)
    .text:08048A45    add     eax, edx              // ((v3^v1^v6 + 1) * v2) + (v3^v1^v6)
    .text:08048A47    and     eax, 0FFFFh           // (((v3^v1^v6 + 1) * v2) + (v3^v1^v6))&0xffff
    .text:08048A4C    mov     [ebp+var_24], eax     // v6 = (((v3^v1^v6 + 1) * v2) + (v3^v1^v6))&0xffff
    .text:08048A4F    mov     eax, [ebp+var_28]
    .text:08048A52    shl     eax, 2
    .text:08048A55    mov     edx, eax
    .text:08048A57    mov     eax, [ebp+s]          
    .text:08048A5A    add     eax, edx
    .text:08048A5C    sub     esp, 4
    .text:08048A5F    push    [ebp+var_24]
    .text:08048A62    push    offset format   ; "%04X"
    .text:08048A67    push    eax             ; s
    .text:08048A68    call    _sprintf              // s[4*i]
    .text:08048A6D    add     esp, 10h
    .text:08048A70    add     [ebp+var_28], 1
    

다음 코드를 통해 브루트포싱을 진행

.. code-block:: python

    from pwn import *
    from ctypes import *
     
    key = "1234567890123456789012345678901234567890123456789012345678901234"
    correct = "F05664E983F54E5FA6D5D4FFC5BF930743F60D8FC2C78AFBB0AF7C82664F2043"
     
    libc = CDLL("libc.so.6")
     
    p=process("./secure_login")
    #p=remote('securelogin.acebear.site', 5001)
    seed = libc.time(0)
    libc.srand(seed)
    h = 0   
    pw = '' 
    for n in range(16):
        r = libc.rand()
        for l in range(0x10000):
            if (int(key[4*n:4*n+4],16)*((h^r^l)+1)+(h^r^l))&0xffff==int(correct[4*n:4*n+4],16):
                pw += "%04X" % l
                h = int(correct[4*n:4*n+4],16)
                break   
     
    print p.sendlineafter('name: ','joizel'),
    print p.sendlineafter(': ', pw)
    print p.recv(1024)
     
    p.interactive()
    
