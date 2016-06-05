================================================================================
[trendmicro] poison ivy pcap
================================================================================

poison ivy pcap 복호화 하기

Poison Ivy관련 pcap 덤프 내용 중에 key를 찾으라는 문제인거 같네요. Poison Ivy는 Gh0st RAT과 같이 유명한 원격제어 관리 툴입니다. 이걸 악용해서 APT 공격에 자주 쓰이고 있죠.
pcap을 열어보니 ssl로 encrypt되어 있는데 이걸 어떻게 풀지라는 의문이 들었습니다.
구글신에게 물어보니 poison ivy관련 디코딩 툴이 있다고 합니다. chopshop이라는 툴이네요.

일단 무작정 설치했습니다. 그리고 나서 설명에 나온 대로 해봤는데 계속 so를 찾을 수 없다고 에러가 뜨길래 포기했었습니다.

.. code-block:: bash

    $ chopshop -f net.pcap "poisonivy_23x"

    Warning Legacy Module poisonivy_23x!
    Starting ChopShop (Created by MITRE)
    Initializing Modules ...
            Initializing module 'poisonivy_23x'
                    poisonivy_23x init failure: Couldn't locate camellia.so
    No modules
    ChopShop Complete

(여기까지 하고 별 출력되는 게 없어서 포기했었습니다.)
※ 좀 더 적극적으로 봤다면 풀지 않았을 까 하는 아쉬움이 드네요.

풀이를 보니 poisonivy_23x 모듈을 주고 그 뒤에 camellia.so 위치를 지정해 주어야 하더군요.

.. code-block:: bash

    -p: the path to the required lib file (camellia.so)
    -c: save screen/webcam/audio/key captures to files
    -s: Location to save carved files

    $ chopshop -f net.pcap "poisonivy_23x -c -p 'camellia.so 위치'" -s out/

out폴더에 bmp파일을 확인해보니 패스워드가 나오네요. 
