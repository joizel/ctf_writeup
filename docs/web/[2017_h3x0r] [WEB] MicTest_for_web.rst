======================================================
[2017_h3x0r] [WEB] MicTest for web
======================================================

문제 내용
======================================================

Mic Test. Get flag!

http://13.124.1.51/h3x0rctf/chall1/


문제 풀이
======================================================

해당 사이트에 접속해서 소스보기를 해보면 다음과 같은 base64로 주석처리된 내용이 보인다.

.. code-block:: html

    <!--
    dmlldyAvaDN4MHJjdGYvY2hhbGwxL2Nzcy9mbGFnLmNzcw==-->


base64 decode 하면 다음과 같은 내용이 출력되고, 해당 페이지에 접속하면 플래그를 획득할 수 있다.

.. code-block:: console

    $ echo "dmlldyAvaDN4MHJjdGYvY2hhbGwxL2Nzcy9mbGFnLmNzcw=="|base64 -d
    view /h3x0rctf/chall1/css/flag.css

