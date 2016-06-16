================================================================================================================
[webhacking.kr] 01
================================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph G {
        rankdir="LR";
        node[shape="point"];
        edge[arrowhead="none"]

        {
            rank="same";
            "client"[shape="plaintext"];
            "client" -> step0 -> step1 -> step2;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step3 -> step4 -> step5;
        }
        step0 -> step3[label="5<$_COOKIE[user_lv]<6",arrowhead="normal"];
        step4 -> step1[label="solve",arrowhead="normal"];
    }

|

Source analysis
================================================================================================================

Cookie 값만 만족하면 문제가 해결될 수 있습니다.

.. code-block:: php

    <?
    if(!$_COOKIE[user_lv])
    {
        SetCookie("user_lv","1");
        echo("<meta http-equiv=refresh content=0>");
    }
    ?>
    <html>
    <head>
    <title>Challenge 1</title>
    </head>
    <body bgcolor=black>
    <center>
    <br><br><br><br><br>
    <font color=white>
    ---------------------<br>
    <?

    $password="????";
    if(eregi("[^0-9,.]",$_COOKIE[user_lv])) 
        $_COOKIE[user_lv]=1;
    if($_COOKIE[user_lv]>=6) 
        $_COOKIE[user_lv]=1;
    if($_COOKIE[user_lv]>5) 
        @solve();
    echo("<br>level : $_COOKIE[user_lv]");
    ?>
    <br>
    <pre>
    <a onclick=location.href='index.phps'>----- index.phps -----</a>
    </body>
    </html>

|

Request
================================================================================================================

$_COOKIE[user_lv]이 5보다 크고 6보다 작아야 하기 때문에 5.1값을 넣어주면 문제가 해결됩니다.

