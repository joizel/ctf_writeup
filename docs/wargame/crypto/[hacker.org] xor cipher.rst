============================================================================================================
[hacker.org] XOR Cipher
============================================================================================================

XOR Cipher1
============================================================================================================

.. code-block:: python

    _encode = '3d2e212b20226f3c2a2a2b'.decode('hex')

    text = ''
    for l in _encode:
        m = l.encode('hex')
        m = int(m,16)^79
        text += chr(m)

    print text

|

XOR Cipher2
============================================================================================================

.. code-block:: python

    _encode = '948881859781c4979186898d90c4c68c85878f85808b8b808881c6c4828b96c4908c8d97c4878c858888818a8381'.decode('hex')

    for x in range(255):
        text = ''

        for l in _encode:
            m = l.encode('hex')
            m = int(m,16)^x
            if 31<m and m<128:
                text += chr(m)

        if len(text)==len(_encode):
            print text

|

XOR Cipher3
============================================================================================================


.. code-block:: python

    _encode = '31cf55aa0c91fb6fcb33f34793fe00c72ebc4c88fd57dc6ba71e71b759d83588'.decode('hex')

    for x in range(255):
        for y in range(255):
            tmp = x
            text = ''

            for l in _encode:
                m = l.encode('hex')
                m = tmp ^ int(m,16)
                text += chr(m)
                tmp = (tmp+y)%256

            print text



|

XOR Long Cipher
============================================================================================================

- 키 길이 4바이트
- 평문 첫 바이트는 키 첫번째 바이트로 암호화되어 있음
- 평문 다섯번째 바이트는 키 첫번째 바이트로 암호화를 얻을 수 있음

.. code-block:: python

    _encode = '8776459cf37d459fbb7b5ecfbb7f5fcfb23e478aaa3e4389f378439aa13e4e96a77b5fc1f358439df36a4486a03e4381b63e5580a66c0c8ebd6d5b8aa13e459cf34e4d9fa67f02cf90714288a17f589abf7f5886bc705fcfbc700c96bc6b5ecfb7775f8cbc68499daa3f'.decode('hex')
    _list = ''
    _key = 'd31e2cef'.decode('hex')

    for num in range(len(_encode)/4):
        _list += _encode[4*num+3]

    for x in range(255):
        text = ''

        for l in _list:
            m = l.encode('hex')
            m = int(m,16)^x
            if 31<m and m<128 and m!=36 and m!=92 and m!=59:
                text += chr(m)

        if len(text)==len(_list):
            print x
            print text
papua

    # 211 T hhay rt  seunr uCrlooodo
    # 30  hcea  f eFt  rs Paoaannuiv
    # 44  iirskoobsohoy wia.ntts rse
    # 239 sp  efuy.rinoaesp gui y cr



|


Feedback Cipher
============================================================================================================

.. code-block:: python

    _encode = '751a6f1d3d5c3241365321016c05620a7e5e34413246660461412e5a2e412c49254a24'.decode('hex')

    text = ''

    for l in range(len(_encode)-1):
        x = _encode[l].encode('hex')
        y = _encode[l+1].encode('hex')
        m = int(x,16)^int(y,16)
        text += chr(m)

    print text

|


Feedback Cipher 2
============================================================================================================

.. code-block:: python

    _encode = '310a7718781f734c31425e775a314f3b40132c5122720599b2dfb790fd8ff894add2a4bdc5d1a6e987a0ed8eee94adcfbb94ee88f382127819623a404d3f'.decode('hex')


    for x in range(255):
        k = (0x3f+x)%0x100
        text = ''
        for l in _encode:
            l = l.encode('hex')
            l = int(l,16)
            m = l ^ k
            k = (l + x) % 0x100        
            if 31<m and m<128:
                #print chr(m)
                text += chr(m)

        if len(text)+1==len(_encode):
            print text


|

base7 convert
============================================================================================================

.. code-block:: python

    a = 28679718602997181072337614380936720482949

    def convert(n, base):
        T = "0123456789ABCDEF"
        q, r = divmod(n, base)
        if q == 0:
            return T[r]
        else:
            return convert(q, base) + T[r]

    print convert(a, 7)

|

