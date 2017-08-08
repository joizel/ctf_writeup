============================================================================================================
[hack-me] neutron star
============================================================================================================


.. code-block:: python

    from PIL import Image

    img = Image.open('prob7_1DDE562BA7F6A03EB39F42F76BF05098.php')
    img = img.convert('RGB')

    w = img.size[0]
    h = img.size[1]

    cnt = 0
    ans = ""
    temp = 0

    for i in range(h):
        for j in range(w):
            r, g, b = img.getpixel((j, i))
            if r==250 and g==255 and b==240:
                cnt+=1
                if cnt%2==1:
                    ans += chr(j)
                    temp = j
                else:
                    ans += chr(j - temp)
                    temp = 0
    print ans

