============================================================================================================
[reversing] JavaCrackMe
============================================================================================================

Flow Chart
================================================================================================================

.. graphviz::

    digraph foo {
        a -> b -> c;
        c -> d [label="1",arrowhead="normal"];
        c -> e [label="0",arrowhead="normal"];
        
        a [shape=box, color=lightblue, label="IDA"];
        b [shape=box, label="main()"];
        c [shape=diamond, label="sub_8048451()"];
        d [shape=box, label="correct"];
        e [shape=box, label="wrong"];
        
        
    }

|

JAD
============================================================================================================

jar 압축을 푼 뒤 class 파일에 대해 java decompile을 하면 jad 파일이 나옵니다.

JavaCrackMe
------------------------------------------------------------------------------------------------------------

.. code-block:: java

    import java.io.PrintStream;

    public class JavaCrackMe
    {

        public JavaCrackMe()
        {
        }

        public static final synchronized volatile transient void main(String args[])
        {
            try
            {
                System.out.println("Reversing.Kr CrackMe!!");
                System.out.println("-----------------------------");
                System.out.println("The idea came out of the warsaw's crackme");
                System.out.println("-----------------------------\n");
                long l = Long.decode(args[0]).longValue();
                l *= 26729L;
                if(l == 0xeaaeb43e477b8487L)
                    System.out.println("Correct!");
                else
                    System.out.println("Wrong");
            }
            catch(Exception exception)
            {
                System.out.println("Please enter a 64bit signed int");
            }
        }
    }

|


Integer Overflow
------------------------------------------------------------------------------------------------------------

어떤 숫자에 26729를 곱한 값이 0xeaaeb43e477b8487과 같으면 Correct가 됩니다.

0xeaaeb43e477b8487을 26729로 나누면 나누어지지 않으므로 Integer Overflow를 이용해야합니다.

