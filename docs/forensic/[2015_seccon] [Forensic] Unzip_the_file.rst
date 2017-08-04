=================================================================================
[2015_seccon] [Forensic] Unzip the file
=================================================================================



`pkcrack`_ 을 이용한 패스워드 크랙

1. 압축하기 이 전 원본 파일을 찾는다.

2. 원본 파일과 압축된 파일을 비교하여 key값을 찾는다.

3. key값을 이용하여 압축을 푼 파일을 찾는다.


.. code-block:: bash

    $ unzip -v unzip.zip

    Archive:  unzip.zip
     Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
    --------  ------  ------- ---- ---------- ----- --------  ----
       14182  Defl:N     5288  63% 2015-11-30 16:23 30b7a083  backnumber08.txt
       12064  Defl:N     4839  60% 2015-11-30 16:22 b93d9a46  backnumber09.txt
       22560  Defl:N    11021  51% 2015-12-01 15:21 fcd63eb6  flag
    --------          -------  ---                            -------
       48806            21148  57%                            3 files


    $ ./pkcrack-1.2.2/src/extract unzip.zip backnumber08.txt enc.txt

    $ wget http://2014.seccon.jp/mailmagazine/backnumber08.txt

    $ wget http://2014.seccon.jp/mailmagazine/backnumber09.txt

    $ zip enc.zip backnumber08.txt backnumber09.txt
      adding: backnumber08.txt (deflated 63%)
      adding: backnumber09.txt (deflated 60%)

    $ ./pkcrack-1.2.2/src/extract enc.zip backnumber08.txt plain.txt

    $ ./pkcrack-1.2.2/src/pkcrack -c enc.txt -p plain.txt
    Files read. Starting stage 1 on Tue Nov  3 07:19:07 2015
    Generating 1st generation of possible key2_5299 values...done.
    Found 4194304 possible key2-values.
    Now we're trying to reduce these...
    Lowest number: 984 values at offset 970
    Lowest number: 932 values at offset 969
    Lowest number: 931 values at offset 967
    Lowest number: 911 values at offset 966
    Lowest number: 906 values at offset 965
    Lowest number: 904 values at offset 959
    Lowest number: 896 values at offset 955
    Lowest number: 826 values at offset 954
    Lowest number: 784 values at offset 606
    Lowest number: 753 values at offset 206
    Done. Left with 753 possible Values. bestOffset is 206.
    Stage 1 completed. Starting stage 2 on Tue Nov  3 07:19:16 2015
    Ta-daaaaa! key0=270293cd, key1=b1496a17, key2=8fd0945a
    Probabilistic test succeeded for 5098 bytes.
    Ta-daaaaa! key0=270293cd, key1=b1496a17, key2=8fd0945a
    Probabilistic test succeeded for 5098 bytes.

    $ ./pkcrack-1.2.2/src/zipdecrypt 270293cd b1496a17 8fd0945a unzip.zip out.zip
    
    $ unzip out.zip

.. _`pkcrack`: https://www.unix-ag.uni-kl.de/~conrad/krypto/pkcrack.html
