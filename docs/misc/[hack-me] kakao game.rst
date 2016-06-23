============================================================================================================
[hack-me] kakao game
============================================================================================================


.. code-block:: python

    import dpkt
    import base64

    cap_file = open('./kakao game.pcap','rb')
    test_cap = dpkt.pcap.Reader(cap_file)

    for ts,buf in test_cap:
        eth = dpkt.ethernet.Ethernet(buf)
        ip = eth.data
        tcp = ip.data
        if tcp.dport == 80 and len(tcp.data) > 0:
            try:
                http_req = tcp.data
                print http_req
            except:
                continue

        if tcp.sport == 80 and len(tcp.data) > 0:
            try:
                http_res = dpkt.http.Response(tcp.data)
                #print http_res.body
                if 'eyJzd' in http_res.body:
                    print base64.b64decode(http_res.body)

            except:
                continue

