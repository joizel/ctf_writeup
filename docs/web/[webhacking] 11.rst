================================================================================================================
[webhacking.kr] 11
================================================================================================================

1. 인젝션 취약점이 존재할 수 있는 입력부분을 찾아 인젝션 존재 여부를 확인한다.

접속하면 아래와 같은 메시지와 함께 wrong이란 메시지가 출력된다.

.. code-block:: json

    wrong

    $pat="/[1-3][a-f]{5}_.*125.140.118.254.*\tp\ta\ts\ts/";
    if(preg_match($pat,$_GET[val])) { echo("Password is ????"); }

해당 조건을 만족해서 request를 날리면 통과한다.

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/codeing/code2.html?val=1aaaab_a125.140.118.254a\tp\ta\ts\ts/"
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    r = requests.get(url, cookies=cookies, verify=False)

    print r.content

