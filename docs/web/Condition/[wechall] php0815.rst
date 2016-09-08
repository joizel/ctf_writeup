================================================================================================================
[wechall] PHP8018
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

    <?
    # Only allow these ID's
    $whitelist = array(1, 2, 3);

    # if show is not set die with error.
    if (false === ($show = isset($_GET['show']) ? $_GET['show'] : false)) {
        die('MISSING PARAMETER; USE foo.bar?show=[1-3]');
    }
    # check if get var is sane (is it in whitelist ?)
    elseif (in_array($show, $whitelist))
    {
        $query = "SELECT 1 FROM `table` WHERE `id`=$show";
        echo 'Query: '.htmlspecialchars($query, ENT_QUOTES).'<br/>';
        die('SHOWING NUMBER '.htmlspecialchars($show, ENT_QUOTES));
    }
    else # Not in whitelist !
    {
        die('HACKER NONONO');
    }
    ?>

|

solve
================================================================================================================


- 3735929054의 16진수 값