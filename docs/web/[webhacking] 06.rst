================================================================================================================
[webhacking.kr] 06
================================================================================================================

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
        step0 -> step1[label="join.php",arrowhead="normal"];
        step3 -> step2[label="javascript Obfuscated",arrowhead="normal"];
        step4 -> step5[label="id= admin&pw=1234",arrowhead="normal"];
        step7 -> step6[label="solve",arrowhead="normal"];
    }

|

Source analysis
================================================================================================================

입력 부분: $_COOKIE[user], $_COOKIE[password]

출력 부분: @solve(6,100);

.. code-block:: php
    
    <?php 
    if(!$_COOKIE[user]) 
    { 
        $val_id="guest"; 
        $val_pw="123qwe"; 

        for($i=0;$i<20;$i++) 
        { 
            $val_id=base64_encode($val_id); 
            $val_pw=base64_encode($val_pw); 

        } 

        $val_id=str_replace("1","!",$val_id); 
        $val_id=str_replace("2","@",$val_id); 
        $val_id=str_replace("3","$",$val_id); 
        $val_id=str_replace("4","^",$val_id); 
        $val_id=str_replace("5","&",$val_id); 
        $val_id=str_replace("6","*",$val_id); 
        $val_id=str_replace("7","(",$val_id); 
        $val_id=str_replace("8",")",$val_id); 

        $val_pw=str_replace("1","!",$val_pw); 
        $val_pw=str_replace("2","@",$val_pw); 
        $val_pw=str_replace("3","$",$val_pw); 
        $val_pw=str_replace("4","^",$val_pw); 
        $val_pw=str_replace("5","&",$val_pw); 
        $val_pw=str_replace("6","*",$val_pw); 
        $val_pw=str_replace("7","(",$val_pw); 
        $val_pw=str_replace("8",")",$val_pw); 

        Setcookie("user",$val_id); 
        Setcookie("password",$val_pw); 

        echo("<meta http-equiv=refresh content=0>"); 
    } 
    ?> 

    <html> 
    <head> 
    <title>Challenge 6</title> 
    <style type="text/css"> 
    body { background:black; color:white; font-size:10pt; } 
    </style> 
    </head> 
    <body> 

    <? 
        $decode_id=$_COOKIE[user]; 
        $decode_pw=$_COOKIE[password]; 

        $decode_id=str_replace("!","1",$decode_id); 
        $decode_id=str_replace("@","2",$decode_id); 
        $decode_id=str_replace("$","3",$decode_id); 
        $decode_id=str_replace("^","4",$decode_id); 
        $decode_id=str_replace("&","5",$decode_id); 
        $decode_id=str_replace("*","6",$decode_id); 
        $decode_id=str_replace("(","7",$decode_id); 
        $decode_id=str_replace(")","8",$decode_id); 

        $decode_pw=str_replace("!","1",$decode_pw); 
        $decode_pw=str_replace("@","2",$decode_pw); 
        $decode_pw=str_replace("$","3",$decode_pw); 
        $decode_pw=str_replace("^","4",$decode_pw); 
        $decode_pw=str_replace("&","5",$decode_pw); 
        $decode_pw=str_replace("*","6",$decode_pw); 
        $decode_pw=str_replace("(","7",$decode_pw); 
        $decode_pw=str_replace(")","8",$decode_pw); 

        for($i=0;$i<20;$i++) 
        { 
            $decode_id=base64_decode($decode_id); 
            $decode_pw=base64_decode($decode_pw); 
        } 

        echo("<font style=background:silver;color:black>&nbsp;&nbsp;HINT : base64&nbsp;&nbsp;</font><hr><a href=index.phps style=color:yellow;>index.phps</a><br><br>"); 
        echo("ID : $decode_id<br>PW : $decode_pw<hr>"); 

        if($decode_id=="admin" && $decode_pw=="admin") 
        { 
            @solve(6,100); 
        } 
    ?> 
    </body> 
    </html> 


|

Cookie Base64 decode
================================================================================================================

입력 부분이 쿠키값이기 때문에 쿠키값을 확인해보면 쿠키값이 존재함을 확인할 수 있다.
해당 쿠키값을 base64로 디코드했을 때 id는 guest이고 pw는 123qwe으로 보인다.
출력 부분을 보면 id와 pw가 admin일 경우 해결이 된다고 하니 admin을 encode해보자.

.. code-block:: python

    import base64

    user = 'admin'
    pw = 'admin'

    for i in range(20):
        user = base64.b64encode(user)
        pw = base64.b64encode(pw)

    user=user.replace("1","!")
    user=user.replace("2","@")
    user=user.replace("3","$")
    user=user.replace("4","^")
    user=user.replace("5","&")
    user=user.replace("6","*")
    user=user.replace("7","(")
    user=user.replace("8",")")

    pw=pw.replace("1","!") 
    pw=pw.replace("2","@") 
    pw=pw.replace("3","$") 
    pw=pw.replace("4","^") 
    pw=pw.replace("5","&") 
    pw=pw.replace("6","*") 
    pw=pw.replace("7","(") 
    pw=pw.replace("8",")") 

    print user
    print pw