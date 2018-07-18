==============================================================
[2018_angstromCTF] [REV] Product Key
==============================================================

문제내용
==============================================================

Artemis wants a copy of Windows, but she doesn’t feel like paying for it. She decided to hack Microsoft’s servers to generate a product key, and found their verification software, which runs on Linux for some reason. Can you get her a working product key (form ABCD-EFGH-IIJK-LMNO-PQRS-TUVW, each uppercase letter is a digit) using the email artemis.tosini@example.com and name Artemis Tosini?

문제 풀이
==============================================================

먼저 바이너리 파일을 리눅스에서 실행해보면 다음과 같다.

.. code-block:: sh

    $ ./activate
    Name: Artemis Tosini
    Email: artemis.tosini@example.com
    Product key: 1234-5678-9012-3456-7890-1234
    Invalid product key

Name과 Email과 Product key 입력값으로 하여 일치여부를 확인하는 문제이다.
문제에서 Name과 Email은 주어졌으며 Prodcuct key를 구하면 해결할 수 있다.

|

IDA hexray decompiler
---------------------------------------------------------------------

입력한 값을 verify_key라는 함수를 통해 인증하여, 조건이 만족할 경우 참이되어 "Windows has been activated" 메시지를 출력한다.

.. code-block:: c

    if ( (unsigned __int8)verify_key(s_name, v6_email, v7_key) )
    {
        puts("Windows has been activated");
        result = 0;
    }

만족해야할 출력값 조건은 name과 email만으로 계산이 가능하기 때문에 먼저 계산한다.

.. code-block:: python

    def sumChars(a1,a2,a3,a4):
        v5 = 0
        for i in range(a2,a3,a4):
            v5 += ord(a1[i])

        return v5


    a2_email = "artemis.tosini@example.com"
    a1_name = "Artemis Tosini"
    pad = "773C1E6B3913220F2402735967642173171E6D5B046665515B4357270E6A0F6D2F0148443B485E804E1F27113346334A255E333228606E00061F2867437D5732".decode("hex")

    v10 = 0
    v22 = []
    v23 = []
    v22.append(a2_email)
    v22.append(a1_name)
    for c in range(2):
        v21 = v22[c]
        d = len(v22[c])
        if d <= 31:
            for e in range(32-d):
                v21+= pad[v10 + e]
            v10 = 32 - d
            #v21+= 0
            v23.append(v21)

    a2_email2 = ""
    a1_name2 = ""
    for f in range(32): 
        a2_email2 += chr(ord(v23[0][f])^5)
        a1_name2 += chr(ord(v23[1][f])^0xf)

    v24 = []
    v37 = []
    v25 = 2
    v26 = 4
    v27 = 6
    v28 = 8
    v29 = 7
    v30 = 5

    v31 = sumChars(a2_email2, 0, 10, 1)
    v32 = sumChars(a2_email2, 10, 25, 1)
    v33 = sumChars(a2_email2, 25, 32, 1)
    v34 = sumChars(a1_name2, 0, 13, 1)
    v35 = sumChars(a1_name2, 13, 20, 1)
    v36 = sumChars(a1_name2, 20, 32, 1)

    v24.append(v25)
    v24.append(v26)
    v24.append(v27)
    v24.append(v28)
    v24.append(v29)
    v24.append(v30)

    v37.append(v31)
    v37.append(v32)
    v37.append(v33)
    v37.append(v34)
    v37.append(v35)
    v37.append(v36)

    for j in range(6):
        v37[j] = v37[j] * v24[j] % 10000

    for k in range(6):
        print v37[k],

    print a2_email2.encode("hex")  
    print a1_name2.encode("hex")  

output이 [2040, 6016, 2964, 504, 2891, 4600]과 같으면 됨.
padding된 email과 name값도 저장


radare2를 통한 디버깅
---------------------------------------------------------------------

hexray decompiler로 출력된 코드 중 이해가 안가는 부분에 대해서는 디버깅으로 계산식을 확인함

.. code-block:: sh

    $ r2 -d ./activate
    Process with PID 9470 started...
    = attach 9470 9470
    bin.baddr 0x00400000
    Using 0x400000
    asm.bits 64
     -- Execute commands on a temporary offset by appending '@ offset' to your command.
    [0x7f6ed7f1bc30]> aaa
    [x] Analyze all flags starting with sym. and entry0 (aa)
    [x] Analyze function calls (aac)
    [x] Analyze len bytes of instructions for references (aar)
    [x] Use -AA or aaaa to perform additional experimental analysis.
    [x] Constructing a function name for fcn.* and sym.func.* functions (aan)
    = attach 9470 9470
    9470
    [0x7f6ed7f1bc30]> dcu main
    Continue until 0x00400ff8 using 1 bpsize
    hit breakpoint at: 400ff8
    [0x00400ff8]>


기본 명령어

.. code-block:: sh
    
    V + p + p (disassembly)
    step into (s)
    step over (S)
    :px 200@[address]
    s 특정 주소: 특정주소까지 이동
    pdj 1: 디스어셈블 내용 json 형식으로 1줄 출력

해당 명령을 python z3 플러그인을 이용해서 실행하면 플래그를 획득할 수 있다.

.. code-block:: python

    import string
    from z3 import *

    a2_email2 = "64777160686c762b716a766c6b6c45607d64687569602b666a6872391b6e3c16".decode("hex")
    a1_name2 = "4e7d7b6a62667c2f5b607c6661662d002b0d7c56686b2e7c181162540b696a5e".decode("hex")
    pad = "773C1E6B3913220F2402735967642173171E6D5B046665515B4357270E6A0F6D2F0148443B485E804E1F27113346334A255E333228606E00061F2867437D5732".decode("hex")

    output = []
    def swapArr(a1,a2,a3):
        a1[a2],a1[a3] = a1[a3],a1[a2]
        return a1

    def sumChars(a1,a2,a3,a4):
        v5 = 0
        for i in range(a2,a3,a4):
            v5 += ord(a1[i])

        return v5


    if __name__=="__main__":
        output = [Int('key[%d]' %i)for i in range(6)]
        result = [2040, 6016, 2964, 504, 2891, 4600]
        s = Solver()

        for g in range(6):        
            v5 = sumChars(a2_email2, 0, 32, (g + 2));
            output[g] -= (sumChars(a2_email2, (g + 1), 32, (g + 2)) * v5) % 10000
            v6 = sumChars(a1_name2, 0, 32, 2)
            output[g] += 4 * (v6 - sumChars(a1_name2, 1, 32, 2))

        swapArr(output, 3, 4)
        swapArr(output, 2, 5)
        swapArr(output, 1, 5)
        swapArr(output, 2, 3)
        swapArr(output, 0, 5)
        swapArr(output, 4, 5)

        for h in range(6):
            output[h] += sumChars(a1_name2, 0, 32, 1)
            output[h] += sumChars(a2_email2, 0, 32, 1)

        for i in range(6):
            v7 = sumChars(a2_email2, (4 * i), (4 * i + 1), 1)
            output[i] += v7 % sumChars(a1_name2, (4 * i + 2), (4 * i + 3), 1)

        for j in range(6):
            s.add(output[j] == result[j] )

        if s.check()==sat:
            print s.model()
