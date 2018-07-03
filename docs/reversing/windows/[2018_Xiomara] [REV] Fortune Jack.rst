=====================================================================
[2018_Xiomara] [REV] Fortune Jack
=====================================================================

문제 내용
=====================================================================

If your smartphone gets connected to a VPN, you feel like you won a lucky draw.

Lucky_Drawer.exe

문제 풀이
=====================================================================

실행 시 로또같은 화면이 나오며, Generate 버튼 클릭 시 가운데 숫자가 변경되며 "Sorry You are Not So Lucky :(  OH!" 메시지 출력
ilspy로 디컴파일한 뒤, 출력되는 문자열을 검색하면 다음 코드 부분을 확인할 수 있음. 

.. code-block:: c#

    // XiomaraChallenge.Form1
    private void checkflag(int key)
    {
        StringBuilder stringBuilder = new StringBuilder();
        string str = "xiomara{";
        string text = "þæþÖîûìèýÖðæüÖíàíÖàýÖ³\u00a0";
        string str2 = "}";
        string text2 = "DB2C17E69713C8604A91AA7A51CBA041";
        for (int i = 0; i < text.Length; i++)
        {
            stringBuilder.Append((char)((int)text[i] ^ key));
        }
        string text3 = stringBuilder.ToString();
        string text4 = Form1.CreateMD5(text3);
        bool flag = text4.Equals(text2);
        if (flag)
        {
            this.label3.Text = str + text3 + str2;
            this.label3.Visible = true;
        }
        bool flag2 = text4 != text2;
        if (flag2)
        {
            this.label3.Text = "Sorry You are Not So Lucky :(  OH!";
            this.label3.Visible = true;
        }
    }

- 해당 문자열이 출력되지 않으려면 flag2가 false여야 함
    - flag2가 false가 되기 위해서는 text4와 text2가 같아야 함.
        - text4와 text2가 같기 위해서는 stringBuilder값을 md5한 값이 "DB2C17E69713C8604A91AA7A51CBA041"와 같아야 함.
        - md5값을 확인하기 위해서는 임의값 입력을 통한 브루트포싱을 진행

.. code-block:: python

    #-*- coding: utf-8 -*-
    from hashlib import md5

    arr = u"þæþÖîûìèýÖðæüÖíàíÖàýÖ³\u00a0"
    md5hash = "db2c17e69713c8604a91aa7a51cba041"
     
    for key in range(256):
        flag = ''
        for b in arr:
            flag += chr(key ^ ord(b))

        if md5hash == md5(flag.decode("utf-8","ignore").encode('utf-8')).hexdigest():
            print("xiomara{" + flag + "}")

