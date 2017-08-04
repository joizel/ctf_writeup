==============================================================
[2017_BITSCTF] [Forensic] Black hole
==============================================================

문제내용
==============================================================

We are trying to study the Black Holes. One of the most controversial theory related to Black Holes is the "Information Loss Paradox". Calculations suggest that physical information could permanently disappear in a black hole. But this violates the quantum theory. To test the hypothesis, we sent our flag encoded in Base64 format towards a Black Hole. With the help of our Hubble Space Telescope, we got some pictures of the Black Hole. See if you can recover the flag from this information.
[번역] 우리는 블랙홀에 대해 연구하려고 합니다. 블랙홀과 관려뇐 가장 논란이 되는 이론 중 하나가 "Information Loss Paradox" 입니다. 계산에 따르면, 물리적 정보가 블랙홀에서는 영구적으로 사라질 수 있습니다. 그러나, 이건은 양자 이론에 위배됩니다. 가설을 테스트하기 위해 우리는 Base64 형식으로 인코딩된 플래그를 블랙홀로 보냈습니다.  Hubble Space Telescope의 도움으로 블랙홀 사진을 얻었습니다. 이 정보에서 플래그를 복구할 수 있는지 확인하십시오.

.. images:: ../_images/0803-3.jpg
    :align: center


문제 풀이
==============================================================

해당 문제의 경우 BITSCTF에 플래그 유형을 알아야 풀 수 있는 문제이다. BITSCTF 플래그 유형은 다음과 같은 접두사를 가지고 있다.

.. code-block:: console

    BITSCTF{
  
문제에서 Base64형식으로 플래그를 보냈다고 했으니, 해당 문자열을 Base64로 인코딩한 값이 이미지 파일 문자열에 존재하는 지 확인해본다.

.. code-block:: console

    $ echo 'BITSCTF{' | base64
    QklUU0NURnsK
  
    $ strings black_hole.jpg | grep 'QklU'
    UQklUQ1RGe1M1IDAwMTQrODF9
  
    $ echo "QklUQ1RGe1M1IDAwMTQrODF9"|base64 -d
    BITCTF{S5 0014+81}
    
