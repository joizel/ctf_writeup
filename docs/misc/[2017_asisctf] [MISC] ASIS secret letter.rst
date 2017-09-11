==============================================================
[2017_asisctf] [MISC] ASIS secret letter
==============================================================

문제내용
==============================================================

The face is the index of the mind, its ASIS secret letter!


문제 풀이
==============================================================

png, jpeg 2개의 이미지 파일이 주어집니다.

이미지 파일을 binwalk로 확인한 결과 하나의 파일에서 TIFF 파일이 존재함을 확인.
파일을 확인해보니 base64로 이루어져 있는 것으로 보여, base64를 진행했더니 powerd by Stéganô 라는 걸 확인할 수 있었음.

.. code-block:: console

    $ binwalk -e 3baa358f6d671e86f17bc4439cc4062e

    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             JPEG image data, JFIF standard 1.01
    30            0x1E            TIFF image data, big-endian, offset of first image directory: 8
    56            0x38            Zlib compressed data, default compression

    $ binwalk -e e07d17ed7d8104590ff3e17bdf052057

    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             PNG image, 4351 x 2812, 8-bit/color RGB, non-interlaced
    41            0x29            Zlib compressed data, default compression


    $ cat  _3baa358f6d671e86f17bc4439cc4062e.extracted/38
    OEorU2pDQWdabkp2YlNCQlUwbFRJSGRwZEdnZ2JHOTJaU3dnY0d4bFlYTmxJR1pwYm1RZ2MyVmpjbVYwSUcxbGMzTmhaMlVnWVc1a0lISmxjR3g1SUhOdmIyNHNJSEJ2ZDJWeVpXUWdZbmtnOEorUmlTQWdVM1REcVdkaGJzTzBJUENma1lnPQ==


    $ cat _3baa358f6d671e86f17bc4439cc4062e.extracted/38|base64 -d |base64 -d
    💌  from ASIS with love, please find secret message and reply soon, powered by 👉  Stéganô 👈


Stéganô로 구글 검색을 해보니 하나의 툴이 검색됨.

https://github.com/cedricbonhomme/Stegano

해당 툴을 다운 받고 실행해보니 플래그가 나옴.. 왜 triangular_numbers를 썻을까? 의문..

.. code-block:: console

    $ python3 Stegano/bin/stegano-lsb-set reveal -i e07d17ed7d8104590ff3e17bdf052057 -g triangular_numbers


