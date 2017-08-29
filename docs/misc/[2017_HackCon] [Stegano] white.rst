==============================================================
[2017_HackCon] [Stegano] White
==============================================================

문제내용
==============================================================

This is a white image. Or is it?

문제 풀이
==============================================================

이미지 파일을 받고 보면 IEND 뒤에 base64데이터가 붙어있는 것을 볼 수 있다.
해당 base64 데이터를 디코딩하면 또다른 이미지 파일이 나온다.

이런식으로 계속해서 이미지 파일을 디코딩하면 이미지 조각들이 나오게 되고,
해당 이미지 조각을 조합하면 플래그를 획득할 수 있다.


.. code-block:: python

    import base64 

    f = open('final.png','rb')
    data = f.read()
    l = 1
    while True:
        if 'IEND' in data:
            data = data.split('IEND')[1][4:]
            data = base64.b64decode(data)
            g = open (str(l)+'.png','wb')

            g.write(data)
            g.close()
            l = l+1

        else:
            False
            break