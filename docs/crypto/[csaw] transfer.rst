============================================================================================================
[csaw] Transfer
============================================================================================================

.. uml::
    
    @startuml

    start

    :nc client;

    :decode morse;

    :decode rot13;

    stop

    @enduml

|

source analysis
============================================================================================================

pcap 내용을 확인해보니 python 코드와 랜덤한 문자열을 확인할 수 있습니다. 아마도 python 코드로 인코딩된 데이터 값이 랜덤한 문자열인 것으로 보입니다.

**python 코드**

.. code-block:: python

	import string
	import random
	from base64 import b64encode, b64decode

	FLAG = 'flag{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}'

	enc_ciphers = ['rot13', 'b64e', 'caesar']
	# dec_ciphers = ['rot13', 'b64d', 'caesard']

	def rot13(s):
		_rot13= string.maketrans(
			"ABCDEFGHIJKLMabcdefghijklmNOPQRSTUVWXYZnopqrstuvwxyz",
			"NOPQRSTUVWXYZnopqrstuvwxyzABCDEFGHIJKLMabcdefghijklm")
		return string.translate(s, _rot13)

	def b64e(s):
		return b64encode(s)

	def caesar(plaintext, shift=3):
		alphabet= string.ascii_lowercase
		shifted_alphabet= alphabet[shift:] + alphabet[:shift]
		table = string.maketrans(alphabet, shifted_alphabet)
		return plaintext.translate(table)

	def encode(pt, cnt=50):
		tmp = '2{}'.format(b64encode(pt))
		for cnt in xrange(cnt):
			c= random.choice(enc_ciphers)
			i= enc_ciphers.index(c) + 1
			_tmp= globals()[c](tmp)
			tmp= '{}{}'.format(i, _tmp)

		return tmp

	if __name__ == '__main__':
		print encode(FLAG, cnt=?)

**random 문자열**

code-block:: console 
	
	2Mk16Sk5iakYxVFZoS1RsWnZXbFZa......(중략)

|

decode
============================================================================================================

1번 문제의 경우 morse 부호를 디코딩해야 하기 때문에 morse_talk를 사용해서 디코딩을 진행하였습니다. 총 50문제이기 때문에 50번 디코딩을 진행하였습니다.

.. code-block:: python

    import morse_talk as mtalk

    def decode_morse():
        #print ru(s,"text:")
        s.send("123\n")
        #print ru(s, "\n")
        rec_data = ru(s, "\n:")
        morse_data = rec_data.split('  decrypted?')[0].split('What is ')[1]
        #print morse_data
        send_data = mtalk.decode(morse_data.split('   ')[0])
        for l in morse_data.split('   ')[1:]:
            send_data += ' ' + mtalk.decode(l)

        #print send_data
        s.send(send_data+'\n')
        return ru(s,'\n')

    for l in range(50):
        decode_morse()

|

decode rot13
============================================================================================================

2번 문제의 경우 rot13 디코딩을 해야 하는데 기호까지 같이 디코딩을 진행하여야 하기 때문에 codecs에 있는 rot13이 아닌 직접 string에 있는 maketrans를 이용해서 디코딩을 진행하였습니다.

.. code-block:: python

    import string

    def decode_morse():
        s = s.replace("'","`")
        rot13 = string.maketrans( 
            '{|}~ !"#$%&`()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZnopqrstuvwxyz', 
            'nopqrstuvwxyz{|}~ !"#$%&`()*+,-./0123NOPQRSTUVWXYZABCDEFGHIJKLMabcdefghijklm')
        result = string.translate(s, rot13)
        return result

    for m in range(51):
        decode_rot13()

