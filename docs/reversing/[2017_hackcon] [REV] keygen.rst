==============================================================
[2017_HackCon] [REV] keygen
==============================================================


문제내용
==============================================================


This proprietary software asks for a key, can you find what it is?
To get the flag send your key to

defcon.org.in:8082

eg:
echo 'your_key' | nc defcon.org.in 8082


문제 풀이
==============================================================

제공해준 파일을 IDA hexray decompiler를 통해 확인해보니 다음과 같았다.

.. code-block:: c


    int __cdecl main(int argc, const char **argv, const char **envp)
    {
        char *s2; // ST10_8@1
        unsigned __int8 *v4; // ST18_8@1
        int v5; // edx@1
        int v6; // edx@1
        size_t v7; // rdx@1

        s2 = (char *)malloc(0x3E8uLL);
        scanf("%s", s2);
        v4 = (unsigned __int8 *)malloc(0x3E8uLL);
        v5 = strlen("firhfgferfibbqlkdfhh");
        HexToBin(s2, v4, v5);
        v6 = strlen("firhfgferfibbqlkdfhh");
        rot13(v4, s2, v6);
        v7 = strlen("firhfgferfibbqlkdfhh");
        if ( !strncmp("firhfgferfibbqlkdfhh", s2, v7) )
            puts("Match");
        else
            puts("Nope");
        return 0;
    }


입력한 값에 HexToBin과 rot13을 한 값이 "firhfgferfibbqlkdfhh"과 일치하면 Match를 출력한다.
"firhfgferfibbqlkdfhh"를 rot13을 한 후 BinToHex를 하면 값이 다음과 같다.

"73766575737473726573766f6f64797871737575"

해당 값을 보내고 nc로 응답을 기다리면 플래그를 획득할 수 있다.

.. code-block:: console

    $ (python -c 'print "73766575737473726573766f6f64797871737575"'|cat -)|nc defcon.org.in 8082


