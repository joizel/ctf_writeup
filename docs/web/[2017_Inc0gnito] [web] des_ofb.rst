==============================================================
[2017_Inc0gnito] [web] des_ofb
==============================================================


문제내용
==============================================================

I implemented my own DES: Output Feedback Mode DES..!!
I heard that the OFB mode is cryptographically safe.
Here is my favorite secret file encrypted by it.

문제 풀이
==============================================================

OFB 운영모드의 DES 인코딩 python 코드와 암호화된 파일이 주어진다.
암호화 파일을 복호화해서 플래그를 찾는게 문제 핵심으로 보인다.

..code-block:: python

    #!/usr/bin/env python

    import pyDes
    from struct import pack, unpack

    def xors(a, b):
        return pack('<Q', unpack('<Q', a)[0] ^ unpack('<Q', b)[0])

    # pyDes do not support OFB mode
    # so I have to implement myself T_T

    class DES_OFB:
        def __init__(self, iv, key):
            self.iv = iv
            self.key = key
        def encrypt(self, data):
            data += '\x00' * (-len(data) % 8)
            ret = ''
            prev = self.iv
            for i in xrange(0, len(data), 8):
                blk = pyDes.des(self.key, pyDes.ECB).encrypt(prev)
                ret += xors(blk, data[i:i+8])
                prev = blk
            return ret

    # I want to be sure that my precious file is not corrupted!!

    legal = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ{}_0123456789'

    with open('flag', 'rb') as f:
        flag = f.read()
        for ch in flag:
            assert ch in legal

    # iv and key are same as plaintext, but anyway it's secret
    # so I think it's absolutely safe :p

    k = DES_OFB(flag[:8], flag[:8])

    with open('flag.enc', 'wb') as f:
        f.write(k.encrypt(flag))


코드에서 보는 바와 같이 DES_OFB의 Key와 IV 값이 동일하다.
또한, 해당 대회의 플래그 형식이 "INC0{"로 시작하기 때문에 3글자만 브포로 찾으면 Key와 IV 값을 찾을 수 있다.

.. code-block:: python

    import itertools

    legal = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_0123456789}'
    flag = 'INC0{'

    f = open('flag.enc','rb')
    enc_data = f.read().encode('hex')

    for l in itertools.product(legal,repeat=3):
        _chr = ''.join(l)
        for ch in _chr:
            try:
                assert ch in legal
            except AssertionError, e:
                pass

        flag2 = flag + _chr
        k = DES_OFB(flag2[:8], flag2[:8])
        if k.encrypt(flag2)[:8] == enc_data[:8]:
            print '# ' + _chr
            break


키값을 찾았으니 브포를 통해 플래그를 획득할 수 있다.

.. code-block:: python

    import itertools

    legal = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_0123456789}'

    f = open('flag.enc','rb')
    enc_data = f.read().encode('hex')

    flag = 'INC0{1t_'
    for m in range(1,21):
        for l in itertools.product(legal,repeat=2):
            _chr = ''.join(l)
            for ch in _chr:
                try:
                    assert ch in legal
                except AssertionError, e:
                    pass

            flag2 = flag + _chr
            print flag2

            n = 4*m
            k = DES_OFB(flag2[:8], flag2[:8])
            if k.encrypt(flag2)[:16+n] == enc_data[:16+n]:
                flag += _chr
                print '# ' + flag
                break
                #n += 4

    print flag
