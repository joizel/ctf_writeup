==============================================================
[2017_hackit] [Forensic] Foren100
==============================================================
 
문제내용
==============================================================

Description: This file was captured from one of the computers at the Internet cafe. We think that the hacker was using this computer at that time. Try to get his secret documents. ( flag format is flag{...} )


문제 풀이
==============================================================

해당 문제에서 제공해주는 pcap을 확인해보면, USB 데이터에 대한 패킷이다. 
HID 패킷 필터링을 위해 다음과 같이 필터링을 통해 데이터를 추출한다.

.. code-block:: console

    $ tshark -r task.pcap -T fields -e usb.capdata -Y "usb.capdata&&usb.bInterfaceClass == 0x03" > usb_capdata.txt


추출한 데이터는 8바이트로 이루어져 있는데, 3번 째 바이트가 입력값을 나타내는 바이트이다.
입력값에 대한 매핑을 통해 어떤 내용을 입력했는지 확인한다.

.. code-block:: python

    mappings = {
        "00":" ",
        "04":"A",
        "05":"B",
        "06":"C",
        "07":"D",
        "08":"E",
        "09":"F",
        "0a":"G",
        "0b":"H",
        "0c":"I",
        "0d":"J",
        "0e":"K",
        "0f":"L",
        "10":"M",
        "11":"N",
        "12":"O",
        "13":"P",
        "14":"Q",
        "15":"R",
        "16":"S",
        "17":"T",
        "18":"U",
        "19":"V",
        "1a":"W",
        "1b":"X",
        "1c":"Y",
        "1d":"Z",
        "1e":"1",
        "1f":"2",
        "20":"3",
        "21":"4",
        "22":"5",
        "23":"6",
        "24":"7",
        "25":"8",
        "26":"9",
        "27":"0",
        "28":"\\n",
        "29":"esc",
        "2a":"back",
        "2b":"tab",
        "2c":"space",
        "2d":"_",
        "2e":"=",
        "2f":"{",
        "30":"}",
        "31":"\\",
        "32":"Non-US",
        "33":";",
        "34":"'",
        "35":"test1",
        "36":",",
        "37":".",
        "38":"/",
        "39":"Capslock",
        "3a":"F1",
        "3b":"F2",
        "3c":"F3",
        "3d":"F4",
        "3e":"F5",
        "3f":"F6",
        "40":"F7",
        "41":"F8",
        "42":"F9",
        "43":"F10",
        "44":"F11",
        "45":"F12",
        "46":"PrintScreen",
        "47":"ScrollLock",
        "48":"Pause",
        "49":"Insert",
        "4a":"Home",
        "4b":"PageUp",
        "4c":"DeleteForward",
        "4d":"End",
        "4e":"PageDown",
        "4f":"RightArrow",
        "50":"LeftArrow",
        "51":"DownArrow",
        "52":"UpArrow",
    }

    f = open("usb_capdata.txt","rb")
    data = f.readlines()
    input_data = ""
    for l in data:
        point = l.split(':')[2]
        print mappings[point]
        input_data += mappings[point]

    print input_data


.. code-block:: text

    W{W4JU},PT}=J5;9=PS73,I
    K3.BN4;6PJIM0{U'H;FKS1S_
    FLAG{K3YB0ARD_SN4KE_2.0}
    B{{E{FU 7D{=.890}'41C4CE
    3'CI.{5=57K9LC82Y41}5QZ3

    