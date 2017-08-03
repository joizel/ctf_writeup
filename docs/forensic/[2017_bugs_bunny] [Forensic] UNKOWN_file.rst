==============================================================
[2017_bugs_bunny] [Forensic] UNKOWN file !!
==============================================================


문제내용
==============================================================

my friend have send me a strange file ,I`m a noob hacker ,could you please solve it for me ?


문제 풀이
==============================================================

다운받은 파일을 확인한 결과 data 형식이다.

.. code-block:: console

    $ file UNKOWN
    file: data

일단 strings로 확인해본 결과 문자열이 거꾸로 된 이미지 파일인 것으로 보인다.

.. code-block:: console

    $ strings -n7 file|grep '\w\.\w\w\w$'
    ......

    WPMIG htiw detaerC
    tnemmoCtXEt
    EMIt
    sYHp
    DGKb
    RDHI


해당 hex 값을 뽑아낸 다음 역으로 출력을 진행하면 플래그가 있는 이미지 파일을 확인할 수 있다.

.. code-block:: console

    $ xxd -p UNKOWN |tr -d '/\n/' > hexdump
    
.. code-block:: python

    f = open('hexdump','rb')
    q = open('flag.png','wb')
    decode_bin = f.read().decode('hex')
    data = decode_bin[::-1]
    print data
    q.write(data)

    f.close()
    q.close()
