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

