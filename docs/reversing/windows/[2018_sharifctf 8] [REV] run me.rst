=====================================================================
[2018_SharifCTF 8] [REV] Run me
=====================================================================

문제 내용
=====================================================================

Run the attached file. If you can, you will capture the flag.

Note: Apply the minimum changes to make the file executable. Then, the mentioned hash function is md5. Be sure to run it on a real Windows OS (not Wine, etc.)

문제 풀이
=====================================================================

정상적으로 실행되지 않는다.
pefile에 Subsystem이 1일 경우 다음과 같이 실행되지 않을 수 있다. 해당 값을 바꿔보자.

+-------------+---+-----------------------------------+
| NATIVE      | 1 | Doesn't require a subsystem       |
|             |   | (such as a device driver)         |
+-------------+---+-----------------------------------+
| Windows GUI | 2 | Runs in the Windows GUI subsystem |
+-------------+---+-----------------------------------+
| WINDOWS_CUI | 3 | Runs in console subsystem         |
+-------------+---+-----------------------------------+