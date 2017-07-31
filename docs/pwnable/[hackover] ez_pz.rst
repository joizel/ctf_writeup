============================================================================================================
[hackover] ez_pz
============================================================================================================

|

Dynamic Analysis
============================================================================================================

.. code-block:: console


    $ ./ez_pz 

                 ___ ____
          ___ __| _ \_  /
         / -_)_ /  _// / 
         \___/__|_| /___|
            lemon squeezy


    Yippie, lets crash: 0xffee6e8c
    Whats your name?
    > joizel

    Welcome joizel!


    $ ./ez_pz 

                 ___ ____
          ___ __| _ \_  /
         / -_)_ /  _// / 
         \___/__|_| /___|
            lemon squeezy


    Yippie, lets crash: 0xffd6415c
    Whats your name?
    > crashme

    Welcome crashme!
    세그멘테이션 오류 (core dumped)

|

Static Analysis
============================================================================================================

먼저 IDA Hexray를 이용하여 해당 프로그램의 소스를 확인해보면 다음과 같습니다.

.. code-block:: c

    int chall()
    {
        size_t v0; // eax@1
        int result; // eax@3
        char s; // [sp+Ch] [bp-40Ch]@1
        _BYTE *v3; // [sp+40Ch] [bp-Ch]@1

        printf("Yippie, lets crash: %p\n", &s);
        printf("Whats your name?\n");
        printf("> ");
        fgets(&s, 1023, stdin);
        v0 = strlen(&s);
        v3 = memchr(&s, 10, v0);
        if ( v3 )
            *v3 = 0;
        printf("\nWelcome %s!\n", &s);
        result = strcmp(&s, "crashme");
        if ( !result )
            result = vuln((unsigned int)&s, 0x400u);
        return result;
    }

|

