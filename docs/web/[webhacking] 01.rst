================================================================================================================
[webhacking.kr] 01
================================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph {
        subgraph cluster_0 {
            label="client";
            1->2;
        }

        subgraph cluster_1 {
            label="server";
            2->3;
        }

        1->2 [label="5<$_COOKIE[user_lv]<6"];
        2->3 [label="solve"];
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

