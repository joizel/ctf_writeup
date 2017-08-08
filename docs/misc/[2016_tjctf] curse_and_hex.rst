=====================================================================
[2016_tjctf] curse and hex
=====================================================================

.. code-block:: python

    from PIL import Image
    im = Image.open("curses_and_hexes.png")
    width, height = im.size
    a=open('out.txt','w')
    for pixel in im.getdata():
        a.write(chr(pixel[0]))
        a.write(chr(pixel[1]))
        a.write(chr(pixel[2]))
    a.close()

