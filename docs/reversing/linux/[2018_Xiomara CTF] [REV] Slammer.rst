==============================================================
[2018_Xiomara CTF] [REV] Slammer
==============================================================

문제내용
==============================================================



문제 풀이
==============================================================

먼저 바이너리 파일을 리눅스에서 실행해보면 다음과 같다.

.. code-block:: sh

    $ ./slammer
    password: abcd
    Wrong!

password를 입력값으로 하여 일치여부를 확인하는 문제이다.

|

IDA hexray decompiler
---------------------------------------------------------------------

hexray decompiler가 정상적으로 동작하지 않았다.



radare2를 통한 디버깅
---------------------------------------------------------------------


.. code-block:: sh
    
    $ r2 -d ./slammer
    [0x00600120]> aaa
    [x] Analyze all flags starting with sym. and entry0 (aa)
    [x] Analyze function calls (aac)
    [x] Analyze len bytes of instructions for references (aar)
    [x] Use -AA or aaaa to perform additional experimental analysis.
    [x] Constructing a function name for fcn.* and sym.func.* functions (aan)
    = attach 2888 2888
    2888
    [0x00600120]> pd 20
    |           ;-- segment.LOAD1:
    |           ;-- rip:
    / (fcn) entry0 347
    |   entry0 (int arg_19h, int arg_7e7e3e7eh);
    |           ; arg int arg_19h @ rbp+0x19
    |           ; arg int arg_7e7e3e7eh @ rbp+0x7e7e3e7e
    |           0x00600120      b801000000     mov eax, 1
    |           0x00600125      bf01000000     mov edi, 1
    |           0x0060012a      48bee8004000.  movabs rsi, 0x4000e8
    |           0x00600134      ba0b000000     mov edx, 0xb                ; 11
    |           0x00600139      0f05           syscall
    |           0x0060013b      4881ec000100.  sub rsp, 0x100
    |           0x00600142      b800000000     mov eax, 0
    |           0x00600147      bf00000000     mov edi, 0
    |           0x0060014c      4889e6         mov rsi, rsp
    |           0x0060014f      ba32000000     mov edx, 0x32               ; '2' ; 50
    |           ; CODE XREF from 0x006000ee (map.home_joizel_slammer.rwx + 238)
    |           0x00600154      0f05           syscall
    |           0x00600156      b823000000     mov eax, 0x23               ; '#' ; 35
    |           0x0060015b      bf06014000     mov edi, 0x400106
    |           0x00600160      31f6           xor esi, esi
    |           0x00600162      0f05           syscall
    |           0x00600164      4889e1         mov rcx, rsp
    |           0x00600167      48ffc9         dec rcx
    |           0x0060016a      48ffc1         inc rcx
    |           0x0060016d      803978         cmp byte [rcx], 0x78        ; [0x78:1]=255 ; 'x' ; 120
    |       ,=< 0x00600170      7427           je 0x600199
    [0x00600120]> s 0x60016d
    [0x0060016d]> pd 2
    |           0x0060016d      803978         cmp byte [rcx], 0x78   ; [0x78:1]=255 ; 'x' ; 120
    |       ,=< 0x00600170      7427           je 0x600199
    [0x0060016d]> dr rax=0x78
    0x00000000 ->0x00000078
    [0x0060016d]> dr rip=0x600199+3
    0x00600120 ->0x0060019c
    [0x0060016d]> s rip
    [0x0060019c]> pd 1
    |           ;-- rip:
    |           0x0060019c      bfb60c0000     mov edi, 0xcb6              ; 3254
    [0x0060019c]> ds 5*0xcb6+5
    [0x0060019c]> s rip
    [0x006001ae]> pd 1
    |       :   ;-- rip:
    |           0x006001b5      803969         cmp byte [rcx], 0x69   ; [0x69:1]=255 ; 'i' ; 105


기본 명령어

.. code-block:: sh
    
    V + p + p (disassembly)
    step into (s)
    step over (S)
    :px 200@[address]
    s 특정 주소: 특정주소까지 이동
    pdj 1: 디스어셈블 내용 json 형식으로 1줄 출력

해당 명령을 python r2pipe 플러그인을 이용해서 실행하면 플래그를 획득할 수 있다.

.. code-block:: python

    import r2pipe

    r=r2pipe.open("./slammer")
    r.cmd("ood 2>/dev/null")
    r.cmd("s 0x60016d")

    while True:
        rax=r.cmdj("pdj 1")[0]["ptr"]
        print(chr(rax))
        if rax==0x7d:
            print()
            break
        r.cmd("dr rax="+str(rax))
        rip=r.cmdj("pdj 2")[1]["jump"]+3
        r.cmd("dr rip="+str(rip))
        r.cmd("s rip")
        edi=r.cmdj("pdj 1")[0]["ptr"]
        r.cmd("ds "+str(5*edi+5))
        r.cmd("s rip")

