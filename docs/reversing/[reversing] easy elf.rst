============================================================================================================
[reversing] Easy ELF
============================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c -> d;
        c -> e;
        
        a [shape=box, color=lightblue, label="IDA"];
        b [shape=box, label="main()"];
        c [shape=diamond, label="sub_8048451()"];
        d [shape=box, label="correct"];
        e [shape=box, label="wrong"];
        c->d [label="1",arrowhead="normal"];
        c->e [label="0",arrowhead="normal"];
    }

|

IDA
============================================================================================================

hexray decompiler로 확인해봅니다.

.. code-block:: c

    int __cdecl main()
    {
        int result; // eax@2

        write(1, "Reversing.Kr Easy ELF\n\n", 0x17u);
        sub_8048434();
        if ( sub_8048451() == 1 )
        {
            sub_80484F7(); // correct
            result = 0;
        }
        else
        {
            write(1, "Wrong\n", 6u);
            result = 0;
        }
        return result;
    }

sub_804851의 리턴 값이 1이면 correct가 되기 때문에 해당 함수 부분을 확인해봅니다.

.. code-block:: c

    int sub_8048451()
    {
        int result; // eax@2

        if ( byte_804A021 == 0x31 )
        {
            byte_804A020 ^= 0x34u;
            byte_804A022 ^= 0x32u;
            byte_804A023 ^= 0x88u;
            if ( byte_804A024 == 0x58 )
            {
                if ( byte_804A025 )
                {
                    result = 0;
                }
                else if ( byte_804A022 == 0x7C )
                {
                    if ( byte_804A020 == 0x78 )
                        result = byte_804A023 == 0xDDu; //
                    else
                        result = 0;
                }
                else
                {
                    result = 0;
                }
            }
            else
            {
                result = 0;
            }
        }
        else
        {
            result = 0;
        }
        return result;
    }

위의 코드를 확인해보면 해당 값이 출력되기 위한 입력 값을 구할 수 있습니다.

.. code-block:: javascript

    byte_804A020 = 0x78^0x34 = 0x4c
    byte_804A021 = 0x31
    byte_804A022 = 0x7C^0x32 = 0x4e
    byte_804A023 = 0xDD^0x88 = 0x55
    byte_804A024 = 0x58

|