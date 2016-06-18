============================================================================================================
[hacker.org] image rgb
============================================================================================================


.. code-block:: python

    import png

    reader = png.Reader('didactrgb.png')
    w, h, pixels, metadata = reader.read_flat()
    for l in pixels:
        print l

    print(int(''.join(map(lambda number: hex(number)[2:], [156, 84, 198])), 16))

|

.. code-block:: python

    import png

    reader = png.Reader('redline.png')
    w, h, pixels, metadata = reader.read_flat()
    answer = ''
    for l in pixels:
        answer += hex(l)

    print answer

|

.. code-block:: python

    import png

    reader = png.Reader('greenline.png')
    w, h, pixels, metadata = reader.read_flat()
    answer = ''
    for l in pixels:
        answer += chr(l)

    print answer
