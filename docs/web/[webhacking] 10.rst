================================================================================================================
[webhacking.kr] 10
================================================================================================================


Source analysis
================================================================================================================


1. 입력부분과 출력 부분을 찾는다.::

.. code-block:: html

    <html>
    <head>
    <title>Challenge 10</title>
    </head>

    <body>
    <hr style=height:100;background:brown;>
    <table border=0 width=900 style=background:gray>
    <tr><td>
    <a id=hackme style="position:relative;left:0;top:0" onclick="this.style.posLeft+=1;if(this.style.posLeft==800)this.href='?go='+this.style.posLeft" onmouseover=this.innerHTML='yOu' onmouseout=this.innerHTML='O'>O</a><br>
    <font style="position:relative;left:800;top:0" color=gold>|<br>|<br>|<br>|<br>buy lotto</font>
    </td></tr>
    </table>
    <hr style=height:100;background:brown;>

    </body>
    </html>

|


td 값 수정
================================================================================================================

this.style.posLeft+=1 부분을 this.style.posLeft=800으로 바꿔주면 클리어 된다.