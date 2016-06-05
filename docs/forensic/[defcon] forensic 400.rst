=====================================================================
[defcon] forensic400
=====================================================================

메모리 덤프 파일 정보 확인
=====================================================================

(실행 화면 시간이 꽤 걸리기에 정지 화면인 줄 착각할 수 있습니다. -d 옵션을 주면 디버깅 화면까지 나와서 좀 덜지루합니다.)

.. code-block:: console

    $ vol.py imageinfo -f  memory.dmp

    Volatility Foundation Volatility Framework 2.5
    INFO    : volatility.debug    : Determining profile based on KDBG search...
              Suggested Profile(s) : Win7SP0x86, Win7SP1x86
                         AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                         AS Layer2 : FileAddressSpace (/home/joizel/memory.dmp)
                          PAE type : PAE
                               DTB : 0x185000L
                              KDBG : 0x82948c28L
              Number of Processors : 1
         Image Type (Service Pack) : 1
                    KPCR for CPU 0 : 0x82949c00L
                 KUSER_SHARED_DATA : 0xffdf0000L
               Image date and time : 2012-05-28 02:57:03 UTC+0000
         Image local date and time : 2012-05-27 22:57:03 -0400

|

프로세스 정보 확인
=====================================================================

위에서 나온 프로파일이 윈도우이므로, pslist로 프로세스 정보에 대해 확인합니다.

.. code-block:: console

    $ vol.py pslist --profile=Win7SP0x86 -f  memory.dmp

    Volatility Foundation Volatility Framework 2.5
    Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit
    ---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
    [......중략......]
    0x8494dd40 rundll32.exe           3916   3440      2       84      1      0 2012-05-28 02:36:37 UTC+0000
    0x8494ed40 chrome.exe             3924   3440      6      209      1      0 2012-05-28 02:36:37 UTC+0000
    0x8495d508 chrome.exe             4004   3440      6      124      1      0 2012-05-28 02:36:41 UTC+0000
    0x85716d40 thunderbird.ex         2372   2176     36      432      1      0 2012-05-28 02:47:24 UTC+0000
    0x849e2540 gpg.exe                2804   2372      0 --------      1      0 2012-05-28 02:50:29 UTC+0000   2012-05-28 02:50:29 UTC+0000
    0x84a16168 gpg-connect-ag         3616   2372      0 --------      1      0 2012-05-28 02:50:29 UTC+0000   2012-05-28 02:50:34 UTC+0000
    0x84a25d40 gpg-agent.exe          4036   3128      1       72      1      0 2012-05-28 02:50:30 UTC+0000
    0x848a8d40 gpg.exe                2360   2372      0 --------      1      0 2012-05-28 02:54:56 UTC+0000   2012-05-28 02:54:56 UTC+0000
    0x84a25030 gpg.exe                2464   2372      0 --------      1      0 2012-05-28 02:54:56 UTC+0000   2012-05-28 02:54:56 UTC+0000
    0x84965c90 pinentry.exe           3284   4036      0 --------      1      0 2012-05-28 02:55:03 UTC+0000   2012-05-28 02:55:13 UTC+0000
    0x84863030 svchost.exe            2872    492      4       40      0      0 2012-05-28 02:55:09 UTC+0000
    0x85709d40 pinentry.exe           2904   4036      0 --------      1      0 2012-05-28 02:55:13 UTC+0000   2012-05-28 02:55:15 UTC+0000
    0x8491e738 pinentry.exe           3252   4036      0 --------      1      0 2012-05-28 02:55:15 UTC+0000   2012-05-28 02:55:24 UTC+0000
    0x84a27af0 pinentry.exe           1856   4036      0 --------      1      0 2012-05-28 02:55:43 UTC+0000   2012-05-28 02:55:55 UTC+0000
    0x849bc030 gpg.exe                2404   2372      0 --------      1      0 2012-05-28 02:56:03 UTC+0000   2012-05-28 02:56:03 UTC+0000
    0x849ea8c0 pinentry.exe           1168   4036      0 --------      1      0 2012-05-28 02:56:10 UTC+0000   2012-05-28 02:56:18 UTC+0000
    0x848636b8 pinentry.exe           3816   4036      0 --------      1      0 2012-05-28 02:56:18 UTC+0000   2012-05-28 02:56:22 UTC+0000
    0x848e1030 pinentry.exe            668   4036      0 --------      1      0 2012-05-28 02:56:22 UTC+0000   2012-05-28 02:56:23 UTC+0000
    0x84997778 pinentry.exe           2576   4036      0 --------      1      0 2012-05-28 02:56:23 UTC+0000   2012-05-28 02:56:27 UTC+0000
    0x84a22a38 pinentry.exe           3692   4036      0 --------      1      0 2012-05-28 02:56:27 UTC+0000   2012-05-28 02:56:30 UTC+0000
    0x849fb620 pinentry.exe           3868   4036      0 --------      1      0 2012-05-28 02:56:30 UTC+0000   2012-05-28 02:56:32 UTC+0000


|

수상한 프로세스 덤프
=====================================================================

무언가 뒷부분에 gpg-agent(4036)와 thunderbird(2372)가 수상합니다. thunderbird는 mozilla에 의해 제작된 메일 관리 프로그램입니다. (저도 사용 중입니다.)

수상한 프로세스를 pid로 따로 덤프를 떠 확인합니다.

.. code-block:: console

    $ vol.py --profile=Win7SP0x86 -f memory.dmp -p 2372 memdump -D out/

    Volatility Foundation Volatility Framework 2.5
    ************************************************************************
    Writing thunderbird.ex [  2372] to 2372.dmp



    joizel@ubuntu:~$ vol.py --profile=Win7SP0x86 -f memory.dmp -p 4036 memdump -D out/

    Volatility Foundation Volatility Framework 2.5
    ************************************************************************
    Writing gpg-agent.exe [  4036] to 4036.dmp

|

덤프 파일 분석
=====================================================================

위에는 명확한 과정이라고 한다면, 여기서부터는 약간의 추론이 들어가는 거 같습니다. 일단 메일 프로그램을 덤프뜬거니까 메일 내용 중에 ctf key값이 있을 것이라는 추측과 그 메일이 gpg로 암호화되어 있을 테니 pgp key로 복호화해야된다는 추측? 일단 이 대회가 defcon이니 strings로 2372.dmp에 defcon을 검색해봅니다.

.. code-block:: console

    $ strings out/2372.dmp |grep -n "defcon"
    
    143015:"Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>"
    143027:      "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>"
    202193:[GNUPG:] USERID_HINT B2B1A673D7A51CC5 Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>
    226203:      "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>"
    227648:      "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>"

왠지 메일 제목인듯 보입니다. 그럼 메일 제목 주위 내용들을 한 번 살펴보겠습니다.

.. code-block:: console

    $ strings out/2372.dmp |grep -n "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>" -B 5

    143022-[GNUPG:] GOOD_PASSPHRASE
    143023-[GNUPG:] ENC_TO 21877E7CEC1B51DB 1 0
    143024-gpg: encrypted with RSA key, ID EC1B51DB
    143025-[GNUPG:] NO_SECKEY 21877E7CEC1B51DB
    143026-gpg: encrypted with 1024-bit RSA key, ID D7A51CC5, created 2012-05-26
    143027:      "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>"


위에서 예상한 바와 같이 gpg를 이용해 암호화를 진행한 것 같습니다.
그렇다면 gpg-agent 덤프 파일에 "PGP MESSAGE" 값이 있는지 찾아봅시다.

.. code-block:: console

    $ strings out/4036.dmp |grep '\-----BEGIN PGP MESSAGE-----' -A 22
     
    [......중략.......]
    -----BEGIN PGP MESSAGE-----
    Charset: ISO-8859-1
    hIwDsrGmc9elHMUBA/9aYQWeLQ9tSBdFK9mNKNZKuJ5KbTNtt4irHXnxqDXhFTgW
    j77y3oFg6v1MKiEFqVJY1dBsmVYVa6N9pL/hJ5jZswSng6j8bZAGj1DxVobgoSDR
    lwXC/UGatkCrB20TvUMlMUgiz3lKFiqwtQBkhvOgAc+NUVpnoyOCkItqx+RV0IUB
    DAMhh3587BtR2wEIAMb9yaOBY17hSr01i4594PYBZlW1P4fdQgoK+DskDQRFoYeQ
    YFlaR1v0pjTGYz8imFF2KVVym83MRElU/BirXavWaWN3oIIROePp82KgnVKUcoKi
    pfFhw5hnHchkhlo4AateQgHBOibknzfZ38jUyqAoY75k5RV42IfZlAlgizSaGdfs
    gZKeeBSkPTH0GEbvDh116PCZEtP3eY7WpbZ+meSp2kooXZ2qjWF6O84BE6YeguDd
    r5cD5AzkwSpV4kjt9tWZCC0o/eUDZ2yXb1PLYrppdX9kChw+Xc6nkp7nJwvARQNv
    o4vAPwP2iibPcttTqsNgRvPUmUstM3Xr20D/sk7SewHWQlEuKSWyMyTdWKNwSU82
    MxBcDAODNV1Wju7q8KYYdfPcPXgsIHF0MNPCKnX6J6gyf9H45ERMsPzWGKnJQaIJ
    gJQLWPUi6pnqOqf+c68JuINTOmhv7W9XyfyNKEHb/zYcZtF46dK8xYSjyIHzR14E
    uzHweaqnPPHo4w==
    =x441
    -----END PGP MESSAGE-----sensitive, do not share"vulcan.ddtek" <vulcan.ddtek@gmail.com> Poseidon <poseidon.ddtek@gmail.com>

다음 출력을 통해 여러가지 메세지들이 보이는데 그 중에 sensitive, do not share 라고 말미에 적혀있는 것이 보입니다. 왠지 이 메시지에 key가 들어있는 메시지인거 같습니다.

그럼 이제 private key를 찾으면 됩니다. 

라이트업을 보니 찾는 방식이 2가지가 있습니다. 

(1) public key의 RSA n 값 hex search로 검색하는 방식
(2) "(defcon ctf quals key)" string search로 검색하는 방식

|

(1)번 방식을 하기 위해 일단 public key를 찾아보았습니다.


.. code-block:: console

    $ strings memory.dmp |grep -n 'PUBLIC KEY'
    
    893504:-----BEGIN PGP PUBLIC KEY BLOCK-----
    893522:-----END PGP PUBLIC KEY BLOCK-----
    2313423:BEGIN PGP PUBLIC KEY BLOCK
    2313430:END PGP PUBLIC KEY BLOCK
     
    $ strings memory.dmp|sed -n '893504,893522p'
    
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Version: GnuPG v2.0.17 (MingW32)
    mI0ET8EtUQEEANXfPR5qcpm+37qy9dKrREx0vYtzzBQR7178Shg9RwEnJGpshFoq
    i2/xmtCfa1LuAXTuI89yE1Iv4YrmQ3DHw0oLBVUi5FqQUVrqY8UaAEptJR+i9Hh+
    IDhMOcP0SfkDS9fMHQ5HCgqwpkgP0MuY1XuNyx42BtGlBIDhxsPpCr6pABEBAAG0
    OlBvc2VpZG9uIChkZWZjb24gY3RmIHF1YWxzIGtleSkgPHBvc2VpZG9uLmRkdGVr
    QGdtYWlsLmNvbT6IvgQTAQIAKAUCT8EtUQIbIwUJACeNAAYLCQgHAwIGFQgCCQoL
    BBYCAwECHgECF4AACgkQZP3N4PucaV5/ZQP/VpSiXViw/x6dWww+4/PP8orn54z0
    4B2+OVCj7BOzxIUQHYl+hZmmRs3lA/ndugpz4MZ4FPitYZFqw0rZVZ+di5UxO0xq
    tURPGieyIkwOWV3HhsCK2FCQMTLWZWzbxgXFVoPJJjemiPLcAnY7xCSydi6XI2Dj
    E4IX1zbF/rLo89e4jQRPwS1RAQQAxdP8WNMW+iXIxf9m5ekTV3JtK1G8MvZ7xvNP
    jNl4n1V9GgXyCr9MR0aLibKYcxXpzRQ3GF7s2Cj3IxoXVT6kscHCh0malnWxFITP
    siVGX+7v2YOIiaqIDLewOhh456Tg6QCJmGb/icazT0oHICNppTMs+NXqH2u+AGiO
    KFMIuoUAEQEAAYilBBgBAgAPBQJPwS1RAhsMBQkAJ40AAAoJEGT9zeD7nGleHG4E
    AJ5iyDGAo7ikY0PEm2h+xdzRfNWxKcbkiVJR6W6kxr/HUZ+5XqPP3g59DwTcJZ3q
    ohdCaaqGkkCGvTart1GNs6ldGZ+J1SSlfXhVl8jbve8NidyZh5Mrxle0Y3lcmvDM
    M/L88kLcIG0mMr+mULg/IJSjerPjVWrplZVgAz6aZKLC
    =3D811y
    -----END PGP PUBLIC KEY BLOCK-----

찾은 public key를 가지고 pgpdump를 통해 RSA n값을 찾고, RSA n값을 hex search를 진행합니다.

.. code-block:: console

    $ sudo pgpdump -i pubkey
    
    Old: Public Key Packet(tag 6)(141 bytes)
            Ver 4 - new
            Public key creation time - Sun May 27 04:21:53 KST 2012
            Pub alg - RSA Encrypt or Sign(pub 1)
            RSA n(1024 bits) - d5 df 3d 1e 6a 72 99 be df ba b2 f5 d2 ab 44 4c 74 bd 8b 73 cc 14 11 ef 5e fc 4a 18 3d 47 01 27 24 6a 6c 84 5a 2a 8b 6f f1 9a d0 9f 6b 52 ee 01 74 ee 23 cf 72 13 52 2f e1 8a e6 43 70 c7 c3 4a 0b 05 55 22 e4 5a 90 51 5a ea 63 c5 1a 00 4a 6d 25 1f a2 f4 78 7e 20 38 4c 39 c3 f4 49 f9 03 4b d7 cc 1d 0e 47 0a 0a b0 a6 48 0f d0 cb 98 d5 7b 8d cb 1e 36 06 d1 a5 04 80 e1 c6 c3 e9 0a be a9
            RSA e(17 bits) - 01 00 01
 
써칭 결과 3개의 pgp 키가 존재하는 데, 2개는 바이너리 형식에 public 키이고, 나머지 하나가 바로 private key입니다.

private key를 import 하고 위의 pgp 메시지를 복호화하면 key가 나옵니다.

.. code-block:: console

    $ sudo gpg --import testkey
    
    gpg: WARNING: unsafe ownership on configuration file `/home/joizel/.gnupg/gpg.conf'
    gpg: key FB9C695E: secret key imported
    gpg: key FB9C695E: "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>" not changed
    gpg: Total number processed: 1
    gpg:              unchanged: 1
    gpg:       secret keys read: 1
    gpg:   secret keys imported: 1
     
    $ sudo gpg -d 1.txt
    
    gpg: WARNING: unsafe ownership on configuration file `/home/joizel/.gnupg/gpg.conf'
    gpg: NOTE: secret key D7A51CC5 expired at Tue 26 Jun 2012 04:21:53 AM KST
    gpg: encrypted with RSA key, ID EC1B51DB
    gpg: encrypted with 1024-bit RSA key, ID D7A51CC5, created 2012-05-26
          "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>"
    * g o a t s e x * g o a t s e x * g o a t s e x *
    g                                               g
    o /     \             \            /    \       o
    a|       |             \          |      |      a
    t|       `.             |         |       :     t
    s`        |             |        \|       |     s
    e \       | /       /  \\\   --__ \\       :    e
    x  \      \/   _--~~          ~--__| \     |    x
    *   \      \_-~                    ~-_\    |    *
    g    \_     \        _.--------.______\|   |    g
    o      \     \______// _ ___ _ (_(__>  \   |    o
    a       \   .  C ___)  ______ (_(____>  |  /    a
    t       /\ |   C ____)/      \ (_____>  |_/     t
    s      / /\|   C_____)       |  (___>   /  \    s
    e     |   (   _C_____)\______/  // _/ /     \   e
    x     |    \  |__   \\_________// (__/       |  x
    *    | \    \____)   `----   --'             |  *
    g    |  \_          ___\       /_          _/ | g
    o   |              /    |     |  \            | o
    a   |             |    /       \  \           | a
    t   |          / /    |         |  \           |t
    s   |         / /      \__/\___/    |          |s
    e  |           /        |    |       |         |e
    x  |          |         |    |       |         |x
    * g o a t s e x * g o a t s e x * g o a t s e x *

    $ sudo gpg -d 2.txt
    
    gpg: WARNING: unsafe ownership on configuration file `/home/joizel/.gnupg/gpg.conf'
    gpg: NOTE: secret key D7A51CC5 expired at Tue 26 Jun 2012 04:21:53 AM KST
    gpg: encrypted with RSA key, ID EC1B51DB
    gpg: encrypted with 1024-bit RSA key, ID D7A51CC5, created 2012-05-26
          "Poseidon (defcon ctf quals key) <poseidon.ddtek@gmail.com>"
    the key is: as it turns out, Phil Zimmermann also likes sheep.
    