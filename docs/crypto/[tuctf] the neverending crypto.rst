============================================================================================================
[tuctf] Neverending Crypto
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

nc client
============================================================================================================

nc를 통해 받은 데이터를 파싱해서 디코딩해야하기 때문에 nc client를 사용해야 합니다.


.. code-block:: python

    import socket

    PORT = 24069
    HOSTNAME = '146.148.102.236'

    def tcp(ip, port):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(( HOSTNAME, PORT ))
        return s

    def ru(s, term):
        o =  ''
        while True:
            t = s.recv(1)
            o += t
            if term in o or len(t)==0:
                return o

    s = tcp(HOSTNAME,PORT)

|

decode morse
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

