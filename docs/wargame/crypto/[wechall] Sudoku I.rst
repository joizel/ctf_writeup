================================================================================================================
[wechall] Sudoku I
================================================================================================================

solve
================================================================================================================

.. code-block:: python

    # sudoku 문제 풀이

    #-*- coding: utf-8 -*-
    from z3 import *
    import math

    sudoku_dict = {}
    s = Solver()

    for x in range(13):
        for y in range(13):
            # 변수 선언
            v=BitVec('v_%d_%d' % (x,y), 13)
            # v값이 1,2,4,8,16,... 이란 의미
            s.add(v&(v-1) == 0)
            sudoku_dict[x,y]=v

    for i in range(13):
        s.add(reduce(lambda a,b:a|b,[sudoku_dict[i,y] for y in range(13)])==0b1111111111111)
        s.add(reduce(lambda a,b:a|b,[sudoku_dict[x,i] for x in range(13)])==0b1111111111111)

    # 1
    s.add(sudoku_dict[0,0]|sudoku_dict[0,1]|sudoku_dict[1,0]|sudoku_dict[1,1]|sudoku_dict[1,2]|sudoku_dict[2,0]|sudoku_dict[2,1]|sudoku_dict[2,2]|sudoku_dict[2,3]|sudoku_dict[3,0]|sudoku_dict[3,1]|sudoku_dict[3,2]|sudoku_dict[4,1]==0b1111111111111)

    # 2
    s.add(sudoku_dict[0,2]|sudoku_dict[0,3]|sudoku_dict[0,4]|sudoku_dict[0,5]|sudoku_dict[1,3]|sudoku_dict[1,4]|sudoku_dict[1,5]|sudoku_dict[2,4]|sudoku_dict[2,5]|sudoku_dict[2,6]|sudoku_dict[3,3]|sudoku_dict[3,4]|sudoku_dict[3,5]==0b1111111111111)

    # 3
    s.add(sudoku_dict[0,6]|sudoku_dict[0,7]|sudoku_dict[0,8]|sudoku_dict[0,9]|sudoku_dict[1,6]|sudoku_dict[1,7]|sudoku_dict[1,8]|sudoku_dict[2,7]|sudoku_dict[3,6]|sudoku_dict[3,7]|sudoku_dict[3,8]|sudoku_dict[4,6]|sudoku_dict[4,8]==0b1111111111111)

    # 4
    s.add(sudoku_dict[0,10]|sudoku_dict[0,11]|sudoku_dict[0,12]|sudoku_dict[1,9]|sudoku_dict[1,10]|sudoku_dict[1,11]|sudoku_dict[1,12]|sudoku_dict[2,8]|sudoku_dict[2,9]|sudoku_dict[2,10]|sudoku_dict[2,11]|sudoku_dict[2,12]|sudoku_dict[3,9]==0b1111111111111)

    # 5
    s.add(sudoku_dict[4,0]|sudoku_dict[4,2]|sudoku_dict[4,3]|sudoku_dict[5,0]|sudoku_dict[5,1]|sudoku_dict[5,2]|sudoku_dict[6,0]|sudoku_dict[6,1]|sudoku_dict[6,2]|sudoku_dict[7,0]|sudoku_dict[7,1]|sudoku_dict[7,2]|sudoku_dict[8,0]==0b1111111111111)

    # 6
    s.add(sudoku_dict[4,4]|sudoku_dict[4,5]|sudoku_dict[5,3]|sudoku_dict[5,4]|sudoku_dict[5,5]|sudoku_dict[6,3]|sudoku_dict[6,4]|sudoku_dict[6,5]|sudoku_dict[7,3]|sudoku_dict[7,4]|sudoku_dict[8,4]|sudoku_dict[8,5]|sudoku_dict[9,5]==0b1111111111111)

    # 7
    s.add(sudoku_dict[4,7]|sudoku_dict[5,6]|sudoku_dict[5,7]|sudoku_dict[5,8]|sudoku_dict[6,6]|sudoku_dict[6,7]|sudoku_dict[6,8]|sudoku_dict[7,5]|sudoku_dict[7,6]|sudoku_dict[7,7]|sudoku_dict[7,8]|sudoku_dict[8,6]|sudoku_dict[8,7]==0b1111111111111)

    # 8
    s.add(sudoku_dict[3,10]|sudoku_dict[3,11]|sudoku_dict[3,12]|sudoku_dict[4,9]|sudoku_dict[4,10]|sudoku_dict[4,11]|sudoku_dict[4,12]|sudoku_dict[5,9]|sudoku_dict[5,10]|sudoku_dict[5,11]|sudoku_dict[5,12]|sudoku_dict[6,9]|sudoku_dict[6,12]==0b1111111111111)

    # 9
    s.add(sudoku_dict[8,1]|sudoku_dict[8,2]|sudoku_dict[9,0]|sudoku_dict[9,1]|sudoku_dict[9,2]|sudoku_dict[10,0]|sudoku_dict[10,1]|sudoku_dict[11,0]|sudoku_dict[11,1]|sudoku_dict[11,2]|sudoku_dict[12,0]|sudoku_dict[12,1]|sudoku_dict[12,2]==0b1111111111111)

    # 10
    s.add(sudoku_dict[8,3]|sudoku_dict[9,3]|sudoku_dict[9,4]|sudoku_dict[10,2]|sudoku_dict[10,3]|sudoku_dict[10,4]|sudoku_dict[10,5]|sudoku_dict[11,3]|sudoku_dict[11,4]|sudoku_dict[11,5]|sudoku_dict[12,3]|sudoku_dict[12,4]|sudoku_dict[12,5]==0b1111111111111)

    # 11
    s.add(sudoku_dict[9,6]|sudoku_dict[9,7]|sudoku_dict[10,6]|sudoku_dict[10,7]|sudoku_dict[10,8]|sudoku_dict[10,9]|sudoku_dict[11,6]|sudoku_dict[11,7]|sudoku_dict[11,8]|sudoku_dict[12,6]|sudoku_dict[12,7]|sudoku_dict[12,8]|sudoku_dict[12,9]==0b1111111111111)

    # 12
    s.add(sudoku_dict[8,8]|sudoku_dict[8,9]|sudoku_dict[8,10]|sudoku_dict[9,8]|sudoku_dict[9,9]|sudoku_dict[9,10]|sudoku_dict[10,10]|sudoku_dict[10,11]|sudoku_dict[11,9]|sudoku_dict[11,10]|sudoku_dict[12,10]|sudoku_dict[12,11]|sudoku_dict[12,12]==0b1111111111111)

    # 13
    s.add(sudoku_dict[6,10]|sudoku_dict[6,11]|sudoku_dict[7,9]|sudoku_dict[7,10]|sudoku_dict[7,11]|sudoku_dict[7,12]|sudoku_dict[8,11]|sudoku_dict[8,12]|sudoku_dict[9,11]|sudoku_dict[9,12]|sudoku_dict[10,12]|sudoku_dict[11,11]|sudoku_dict[11,12]==0b1111111111111)


    sample = [
        0, 0xa, 0, 0, 0xd, 3, 0, 0, 7, 0, 0, 9, 0,
        4, 0, 0, 0, 0, 0, 0, 0xc, 0, 7, 0xb, 0, 0,
        6, 0, 0, 0xd, 0, 0, 0xc, 0, 3, 0, 8, 0, 0,
        0, 0xb, 0xc, 2, 0, 0, 4, 0, 0, 0, 0, 0, 0,
        0, 0, 4, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 7, 0, 5, 8, 0, 3, 0, 0, 0, 0, 0, 0,
        0, 5, 0xb, 0, 0, 0, 2, 0, 0, 0, 0, 0, 0,
        0, 3, 0, 0, 0, 0, 0, 5, 0, 2, 0, 0, 8,
        0, 0, 0, 0, 0xa, 0, 0, 0, 4, 5, 0, 0, 0, 
        0, 0xc, 0, 0, 0, 0, 0, 0, 0, 0xd, 0, 3, 0,
        0xb, 0, 0, 0, 0, 9, 0, 0, 0, 0, 0, 0, 0, 
        0, 4, 0, 0, 0, 2, 0xd, 0, 8, 0, 0, 0, 0,
        0, 0, 0, 0, 5, 6, 0xa, 0, 0, 0, 0, 0, 0
    ]

    def preinit(x, y, value):
        s.add(sudoku_dict[x, y] == 2 ** (value - 1))

    for l,m in enumerate(sample):
        if m!=0:
            preinit(l//13,l%13,m)


    if s.check()==sat:
        model=s.model()
        for x in range(13):
            line = []
            for y in range(13):
                v = int(str(model[sudoku_dict[x, y]]))
                line.append(str(int(math.log(v, 2)) + 1))

            print (' '.join(line))
