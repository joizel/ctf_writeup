=====================================================================
[2018_VolgaCTF] [REV] Crack Me
=====================================================================

문제 내용
=====================================================================

Just do it.


문제 풀이
=====================================================================

파일 다운로드 시 실행 파일 Crackme.exe와 암호화 텍스트 Crackme.txt가 주어진다.
Crackme.exe 실행 시 다음과 같은 화면이 출력된다.

.. code-block:: console

    Incorrect arguments
    Usage: CrackMe.exe <input_file> <output_file> <user_pass_phrase> <mode>
 

ilspy로 디컴파일한 뒤, 출력되는 문자열을 검색하면 다음 코드 부분을 확인할 수 있다.

.. code-block:: c#

    // CrackMe.Program
    private static void Main(string[] args)
    {
        if (args.Length != 4)
        {
            Console.WriteLine("Incorrect arguments");
            Console.WriteLine("Usage: CrackMe.exe <input_file> <output_file> <user_pass_phrase> <mode>");
            return;
        }
        string text = args[0];
        string outputFile = args[1];
        string userPassword = args[2];
        string a = args[3];
        if (!File.Exists(text))
        {
            Console.WriteLine("Input file doesn't exist");
            return;
        }
        CryptoOperation cryptoOperation = new CryptoOperation();
        cryptoOperation.FileName = text;
        cryptoOperation.UserPassword = userPassword;
        if (a == "encrypt")
        {
            Program.Encrypt(cryptoOperation, outputFile);
            return;
        }
        if (!(a == "decrypt"))
        {
            Console.WriteLine("Incorrect mode");
            Console.WriteLine("Use encrypt or decrypt");
            return;
        }
        Program.Decrypt(cryptoOperation, outputFile);
    }

mode 인자값에 따라 암호화 복호화가 가능하며, 주어진 CrackMe.txt파일을 복호화하는게 이번 문제에 핵심으로 보인다.

.. code-block:: c#

    // CrackMe.CryptoOperation
    public byte[] DecryptFile()
    {
        byte[] result = null;
        using (AesCryptoServiceProvider aesCryptoServiceProvider = new AesCryptoServiceProvider())
        {
            aesCryptoServiceProvider.Key = this.UserKey;
            aesCryptoServiceProvider.IV = this.IV;
            try
            {
                ICryptoTransform transform = aesCryptoServiceProvider.CreateDecryptor(aesCryptoServiceProvider.Key, aesCryptoServiceProvider.IV);
                FileInfo fileInfo = new FileInfo(this.FileName);
                using (MemoryStream memoryStream = new MemoryStream(this.ProcessingBytes))
                {
                    using (CryptoStream cryptoStream = new CryptoStream(memoryStream, transform, CryptoStreamMode.Read))
                    {
                        using (BinaryReader binaryReader = new BinaryReader(cryptoStream))
                        {
                            result = binaryReader.ReadBytes((int)fileInfo.Length);
                        }
                    }
                }
            }
            catch
            {
                Console.WriteLine("Failed to decrypt file {0}: wrong password.", this.FileName);
            }
        }
        return result;
    }

aes로 암복호화를 하고 있으며, Userkey값을 키값으로 쓰고 있다.
UserKey값은 3번째 인자값을 다음과 같이 변행하여 적용한다.

.. code-block:: c#

    // CrackMe.CryptoOperation
    public string UserPassword
    {
        set
        {
            this.KeyLength = 16;
            byte[] userKey = MD5.Create().ComputeHash(Encoding.UTF8.GetBytes(value));
            this.UserKey = this.CombineKeys(userKey);
        }
    }

    // CrackMe.CryptoOperation
    private byte[] CombineKeys(byte[] UserKey)
    {
        AppSettings appSettings = new AppSettings();
        byte[] expr_16 = Encoding.UTF8.GetBytes(appSettings.DefaultKey);
        long num = BitConverter.ToInt64(expr_16, 0);
        long num2 = BitConverter.ToInt64(expr_16, 8);
        long num3 = BitConverter.ToInt64(UserKey, 0);
        long num4 = BitConverter.ToInt64(UserKey, 8);
        long num5 = num ^ num3;
        long num6 = num2 ^ num4;
        long num7 = (~num & num3) | (~num3 & num);
        long num8 = (~num2 & num4) | (~num4 & num2);
        int num9 = BitConverter.ToInt32(BitConverter.GetBytes(num5), 0);
        int num10 = BitConverter.ToInt32(BitConverter.GetBytes(num5), 4);
        int num11 = BitConverter.ToInt32(BitConverter.GetBytes(num6), 0);
        int num12 = BitConverter.ToInt32(BitConverter.GetBytes(num6), 4);
        num9 >>= 2;
        num10 >>= 2;
        num9 <<= 1;
        num10 <<= 1;
        num12 = num9 << 1;
        num11 >>= 2;
        num11 = num9 << 1;
        num12 >>= 2;
        if (~(num9 & num12) == (~num9 | ~num12))
        {
            num11 = num10;
            if (~(num9 & num12) == (~num9 | ~num12))
            {
                num10 = num12;
            }
            else
            {
                num12 = num10;
            }
            num9 = ~num12;
        }
        else
        {
            num11 = num9;
            if (~(~num7) == num5 && ~(~num8) == num6)
            {
                num10 = num12;
            }
            else
            {
                num12 = num10;
            }
            num9 = ~num10;
        }
        num9 = ~num9;
        byte[] bytes = BitConverter.GetBytes(num9);
        byte[] bytes2 = BitConverter.GetBytes(num10);
        byte[] bytes3 = BitConverter.GetBytes(num11);
        byte[] bytes4 = BitConverter.GetBytes(num12);
        byte[] array = new byte[16];
        for (int i = 0; i < 4; i++)
        {
            array[i] = bytes[i];
            array[i + 4] = bytes2[i];
            array[i + 8] = bytes3[i];
            array[i + 12] = bytes4[i];
        }
        return array;
    }

위 코드는 보기엔 복잡한 계산식이지만 트릭이 있다. 
계산식 중 "num ^ num3"은 "~num & num3) | (~num3 & num)"과 같으며, "num2 ^ num4"도 역시 "(~num2 & num4) | (~num4 & num2)"과 같다.
그리고 if 조건문이 존재하는데 "(~(num9 & num12) == (~num9 | ~num12))"은 어떤 값이든 항상 참으로 else 문이 필요없다.
그리고 if 조건문 안에 추가 if 조건문이 존재하는데 "(~(num9 & num12) == (~num9 | ~num12))"은 어떤 값이든 항상 거짓으로 else문으로 가게된다.
해당 내용에 맞게 위 코드는 다음과 같이 정리할 수 있다.


.. code-block:: c#

    // CrackMe.CryptoOperation
    private byte[] CombineKeys(byte[] UserKey)
    {
        AppSettings appSettings = new AppSettings();
        byte[] expr_16 = Encoding.UTF8.GetBytes(appSettings.DefaultKey);
        long num = BitConverter.ToInt64(expr_16, 0);
        long num2 = BitConverter.ToInt64(expr_16, 8);
        long num3 = BitConverter.ToInt64(UserKey, 0);
        long num4 = BitConverter.ToInt64(UserKey, 8);
        long num5 = num ^ num3;
        long num6 = num2 ^ num4;
        long num7 = num ^ num3;
        long num8 = num2 ^ num4;
        int num9 = BitConverter.ToInt32(BitConverter.GetBytes(num5), 0);
        int num10 = BitConverter.ToInt32(BitConverter.GetBytes(num5), 4);
        int num11 = BitConverter.ToInt32(BitConverter.GetBytes(num6), 0);
        int num12 = BitConverter.ToInt32(BitConverter.GetBytes(num6), 4);
        num9 >>= 2;
        num10 >>= 2;
        num9 <<= 1;
        num10 <<= 1;
        num12 = num9 << 1;
        num11 >>= 2;
        num11 = num9 << 1;
        num12 >>= 2;
        num11 = num10;
        num12 = num10;
        num9 = ~num12;
        num9 = ~num9;
        byte[] bytes = BitConverter.GetBytes(num9); # num9 = num12
        byte[] bytes2 = BitConverter.GetBytes(num10); # num10 = (BitConverter.ToInt32(BitConverter.GetBytes(BitConverter.ToInt64(expr_16, 0) ^ BitConverter.ToInt64(UserKey, 0)), 4)>>2) << 1
        byte[] bytes3 = BitConverter.GetBytes(num11); # num11 = num10
        byte[] bytes4 = BitConverter.GetBytes(num12); # num12 = num10
        byte[] array = new byte[16];
        for (int i = 0; i < 4; i++)
        {
            array[i] = bytes[i];
            array[i + 4] = bytes2[i];
            array[i + 8] = bytes3[i];
            array[i + 12] = bytes4[i];
        }
        return array;
    }

결국 리턴값 array에 들어갈 값은 4바이트가 연속되는 값들이 들어가게 된다.

.. code-block:: text

    ex>
    31323334 31323334 31323334 31323334
    41424344 41424344 41424344 41424344

해당 키값을 획득하기 위해 브루트포싱 진행
- 오른쪽으로 2번 시프트 연산 왼쪽으로 1번 시프트 연산
    - 오른쪽 1bit는 0
    - 왼쪽 1bit는 0

.. code-block:: python

    from Crypto.Cipher import AES
    import struct

    f = open("CrackMe.txt","rb")
    data = f.read()
    f.close()

    iv = data[:16]
    data = data[16:]

    for i in range(0x10000000,0x80000000,2):
        key = struct.pack('>I',i)
        aes = AES.new(key*4, AES.MODE_CBC, iv)
        dec = aes.decrypt(data)

        if "Volga" in str(dec):
            print(key)
            print(dec)
            break
