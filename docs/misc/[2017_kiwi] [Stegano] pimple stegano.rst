==============================================================
[2017_kiwi] [Stegano] Pimple Stegano
==============================================================

문제내용
==============================================================

For every message that we give, an image is returned. The image should have the message embedded via steganography. The message in the original image will give us the key.

문제 풀이
==============================================================

웹 사이트에 접속하면 이미지 파일이 하나 나오고 msg를 입력하는 창이 뜬다.
msg를 입력하게 되면 점 배치가 계속해서 바뀌는 걸 확인할 수 있었다.

먼저 이미지가 바뀌는 패턴을 이해하기 위해 msg를 1,2,3으로 입력하여 이미지를 비교해보았다.
기본 틀의 rgb들은 변하지 않고 특정 rgb들만 변하는 걸 확인할 수 있었다.

해당 내용을 코드를 통해 추출한 결과 flag를 획득할 수 있었다.

.. code-block:: python 

    import requests
    from PIL import Image
    import string

    im = Image.open("the-troll.png")
    rgb_im = im.convert('RGB')
    width,height = im.size
    newimdata = []
    extract = []
    output = ""
    cnt = 0
    for i in range(width):
        for j in range(height):
            color = rgb_im.getpixel((i,j))
            if color == (3,3,3):
                output += "1"
                cnt += 1
                #print (i,j)

            elif color == (2,2,2):
                output += "0"
                cnt += 1
                #print (i,j)

            elif color == (96,96,96):
                output += "0"
                cnt += 1
                #print (i,j)

            elif color == (97,97,97):
                output += "0"
                cnt += 1
                #print (i,j)

            else:
                pass

            if cnt==8:
                extract.append(output)
                cnt=0
                output=""
                
    print extract
    text = ""
    for q in extract:
        text += chr(int(q,2))

    print text
    