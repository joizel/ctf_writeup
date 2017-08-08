================================================================================
[2015_boston] [MISC] Riverside
================================================================================

challenge.pcapng 파일을 열어 확인해보니 USB 형식입니다.

.. code-block:: console

    $ tshark -r challenge.pcapng usb.bDescriptorType and usb.urb_type==67

       4  0.075077000        12.0 -> host         USB 82 GET DESCRIPTOR Response DEVICE
      24 0.150578000          1.0 -> host         USB 82 GET DESCRIPTOR Response DEVICE
      60 0.047392000          2.0 -> host         USB 82 GET DESCRIPTOR Response DEVICE
      75 0.074061000          1.0 -> host         USB 82 GET DESCRIPTOR Response DEVICE
      94 0.150211000          3.0 -> host         USB 82 GET DESCRIPTOR Response DEVICE
      96 0.150419000          2.0 -> host         USB 82 GET DESCRIPTOR Response DEVICE
      98 0.150447000          1.0 -> host         USB 82 GET DESCRIPTOR Response DEVICE

.. code-block:: console

    $ tshark -r challenge.pcapng usb.bDescriptorType and usb.urb_type==67 -T fields -e usb.bus_id -e usb.device_address -e usb.idVendor -e usb.idProduct
    
      2   12  0x046d  0xc00e # Logitech M-BJ58/M-BJ69 Optical Wheel Mouse
      3   1   0x1d6b  0x0003 # Linux Foundation 3.0 root hub
      1   2   0x8087  0x8000 # Intel Integrated Rate Matching Hub
      1   1   0x1d6b  0x0002 # Linux Foundation 2.0 root hub
      2   3   0x5986  0x0268 # Acer Integrated Camera
      2   2   0x8087  0x07dc # Intel 7260AC Bluetooth
      2   1   0x1d6b  0x0002 # Linux Foundation 2.0 root hub

.. code-block:: console

    $ tshark -r challenge.pcapng 'usb.data_flag=="present (0)"' -T fields -e usb.bus_id -e usb.device_address -e usb.endpoint_number.endpoint | sort | uniq -c

         12 1   1   0
          2 1   1   1
         12 1   2   0
          1 1   2   1
          7 2   1   0
          1 2   1   1
          1 2   12  0
       7608 2   12  1
          1 2   2   0
          3 2   3   0
          9 3   1   0
          1 3   1   1

USB 마우스 코드
