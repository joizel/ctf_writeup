============================================================================================================
[wargame] crack crack crack it 
============================================================================================================

htaccess
============================================================================================================

잊어버린 패스워드를 찾는 문제인데, 패스워드 앞 부분 7글자 **G4HeulB** 이 주어졌습니다. 그리고, 나머지 글자는 슛자와 알파벳 소문자로 이루어졌다는 힌트도 있습니다.

.. code-block:: text

    blueh4g:$1$mXg0v2ao$DoP2Om1NrTvNmuz5fy8LJ1


John The Ripper라는 브루트 포싱 공격 툴을 하여 패스워드를 찾을 수 있습니다.

현재 문자열 앞에 문자열 7글자가 힌트로 제공되었기 때문에, 7글자 이후 문자열을 브루트 포싱하여야 합니다. 

John The Ripper 기능 중 입력 값에 대한 정규 표현식 지원이 없어 python으로 입력 값에 대해 print로 출력을 하고 stdin으로 입력값을 받도록 합니다. (입력값 리스트를 파일로 저장해서 wordlist 옵션을 사용해도 됩니다.)


.. code-block:: python 

    import itertools

    alpha = 'abcdefghijklmnopqrstuvwxyz0123456789'
    MAXLEN = 5

    for l in range(MAXLEN):
        for m in itertools.product(alpha,repeat=n):
            print 'G4HeulB' + ''.join(m)

실행 결과 다음과 같이 패스워드를 출력합니다.

.. code-block:: console

    $ python str_in.py|john -stdin htpasswd 
    
    Warning: detected hash type "md5crypt", but the string is also recognized as "aix-smd5"
    Use the "--format=aix-smd5" option to force loading these as that type instead
    Loaded 1 password hash (md5crypt, crypt(3) $1$ [MD5 128/128 AVX 12x])
    Warning: OpenMP is disabled; a non-OpenMP build may be faster
    Press Ctrl-C to abort, or send SIGUSR1 to john process for status
     
    G4HeulBzca0      (blueh4g)
    1g 0:00:00:35  0.02838g/s 34545p/s 34545c/s 34545C/s G4HeulBzcal..G4HeulBzcbw
    Use the "--show" option to display all of the cracked passwords reliably
    Session completed
    Traceback (most recent call last):
      File "str_in.py", line 8, in <module>
        print 'G4HeulB' + ''.join(m)
    IOError: [Errno 32] Broken pipe

