==============================================================
[2017_bugs_bunny] [Forensic] Give me the Flag !
==============================================================


문제내용
==============================================================

Please helps me to get the flag !


문제 풀이
==============================================================

flag.zip라는 암호가 걸려있는 압축파일과 각국가 이미지가 들어있는 파일을 제공한다.
국가 이미지 파일들 중에 qrcode를 조각된 이미지파일이 존재하며 그들만 따로 추출한다. 
국가 이미지는 RGB 또는 RGBA로 되어 있지만, qrcode는 grayscale로 되어있어 그 부분을 추출한다.

.. code-block:: console

    $ file *|grep 'grayscale'
    ax.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    cx.png:    PNG image data, 60 x 80, 1-bit grayscale, non-interlaced
    dq.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    hj.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    hm.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    im.png:    PNG image data, 60 x 80, 1-bit grayscale, non-interlaced
    mb.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    mj.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    nb.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    nn.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    pr.png:    PNG image data, 80 x 80, 1-bit grayscale, non-interlaced
    qx.png:    PNG image data, 80 x 60, 1-bit grayscale, non-interlaced
    rp.png:    PNG image data, 60 x 60, 1-bit grayscale, non-interlaced
    vi.png:    PNG image data, 80 x 60, 1-bit grayscale, non-interlaced
    xv.png:    PNG image data, 60 x 80, 1-bit grayscale, non-interlaced
    zy.png:    PNG image data, 80 x 60, 1-bit grayscale, non-interlaced


따로 추출한 파일들을 하나의 파일로 합치자.

.. code-block:: console

    $ convert +append ax.png nb.png nn.png cx.png test1.png
    $ convert +append dq.png hj.png hm.png im.png test2.png
    $ convert +append mb.png mj.png pr.png xv.png test3.png
    $ convert +append qx.png vi.png zy.png rp.png test4.png
    $ convert -append test1.png test2.png test3.png test4.png all.png

합친 파일을 qrcode 스캐너로 패스워드 확인
확인된 패스워드로 flag.zip 압축 해제 후 Readme 파일 확인

.. code-block:: console

    01000010 01110101 01100111 01110011 01011111 01000010 01110101 01101110 01101110 01111001 01111011 00110010 01100010 00111001 00110111 00110010 00110110 00110011 01100010 01100101 01100010 00110111 00110000 01100100 00110000 01100110 00110110 00110101 00111001 01100010 01100100 01100010 00111001 00110011 01100011 01100011 00110101 00110010 00111001 00110001 01100100 00110000 01100001 01111101 

python을 통해 해당 플래그 획득

.. code-blcok:: python

    list = "01000010 01110101 01100111 01110011 01011111 01000010 01110101 01101110 01101110 01111001 01111011 00110010 01100010 00111001 00110111 00110010 00110110 00110011 01100010 01100101 01100010 00110111 00110000 01100100 00110000 01100110 00110110 00110101 00111001 01100010 01100100 01100010 00111001 00110011 01100011 01100011 00110101 00110010 00111001 00110001 01100100 00110000 01100001 01111101"

    flag = ""
    for l in list.split(' '):
        flag += chr(int(l,2))

    print flag

