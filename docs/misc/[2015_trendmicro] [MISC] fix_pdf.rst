================================================================================
[2015_trendmicro] [MISC] fix pdf
================================================================================

일단 pdf를 받아서 열어보려고 하니 정상적으로 열리지 않더군요.

exiftool로 pdf파일을 열어보니 Warning 메시지를 띄우네요. Invalid xref table이라고 합니다. 뭔가 레퍼런스가 정상적이지 않은가 봅니다.

.. code-block:: console

    $ exiftool fix_my_pdf.pdf

    ExifTool Version Number         : 9.46
    File Name                       : fix_my_pdf.pdf
    Directory                       : .
    File Size                       : 32 kB
    File Modification Date/Time     : 2015:09:25 23:44:33-04:00
    File Access Date/Time           : 2015:09:29 20:27:16-04:00
    File Inode Change Date/Time     : 2015:09:26 00:19:24-04:00
    File Permissions                : rw-rw-r--
    File Type                       : PDF
    MIME Type                       : application/pdf
    PDF Version                     : 1.3
    Linearized                      : No
    Warning                         : Invalid xref table

pdfextract를 했더니 다음과 같이 6개의 stream 파일과 2개의 이미지 파일을 떨굽니다.

.. code-block:: bash

    $ pdfextract fix_my_pdf.pdf

    [error] Breaking on: "stream\nx\x01\xED..." at offset 0x1581
    [error] Last exception: [Origami::InvalidObjectError] Cannot determine object (no:13,gen:0) type
    Extracted 6 PDF streams to 'fix_my_pdf.dump/streams'.
    Extracted 0 scripts to 'fix_my_pdf.dump/scripts'.
    Extracted 0 attachments to 'fix_my_pdf.dump/attachments'.
    Extracted 0 fonts to 'fix_my_pdf.dump/fonts'.
    Unable to decode image (stream 11). Invalid indexed palette table
    Extracted 2 images to 'fix_my_pdf.dump/images'.

떨군 파일 중에 streams에 있는 내용을 확인해보니 stream_2.dmp에 <xmpGImg:image> 태그에 /9j/로 시작하는 데이터가 들어있습니다.

.. code-block:: html

    <rdf:li rdf:parseType="Resource">
    <xmpGImg:image>/9j/4AAQSkZJRgABAgEASABIAAD/7QAsUGhvdG9zaG9wIDMuMAA4QklNA+0AAAAAABAASAAAAAEA&#xA;AQBIAAAAAQAB/+4ADkFkb2JlAGTAAAAAAf/bAIQABgQEBAUEBgUFBgkGBQYJCwgGBggLDAoKCwoK&#xA;DBAMDAwMDAwQDA4PEA8ODBMTFBQTExwbGxscHx8fHx8fHx8fHwEHBwcNDA0YEBAYGhURFRofHx8f&#xA;Hx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8fHx8f/8AAEQgAdAEAAwER&#xA;AAIRAQMRAf/EAaIAAAAHAQEBAQEAAAAAAAAAAAQFAwIGAQAHCAkKCwEAAgI(....중략...)HZd&#xA;3f8AIC/+/X/7l+Oy7u/5AX/36/8A3L8dl3d/yAv/AL9f/uX47Lu7/kBf/fr/APcvx2Xd3/IC/wDv&#xA;1/8AuX47Lu7/AJAX/wB+v/3L8dl3d/yAv/v1/wDuX47Lu7/kBf8A36//AHL8dl3f/9k=</xmpGImg:image>

이런 경우 base64로 인코딩되어 있는 것으로 &#xA; 값을 다 지우고 base64 decode를 해주면 jpg 파일이 나옵니다.


.. image:: ../_images/DecodedBase64.jpeg
    :align: center
