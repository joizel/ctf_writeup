==============================================================
[2017_bugs_bunny] [Forensic] For80
==============================================================


문제내용
==============================================================

you can`t see me !!


문제 풀이
==============================================================

gif파일이지만 굉장히 얇게 되어 있어 어떤 이미지인지 확인이 어렵다.
imagemagick의 convert 명령을 통해 gif의 움직이는 이미지 파일을 png로 각각 저장한다.

.. code-block:: console

    $ convert -coalesce task.gif gif/out%05d.png

저장한 파일을 하나의 이미지 파일로 합치면 플래그를 확인할 수 있다.

.. code-block:: console

    $ convert +append $(find gif|sort) task.png
