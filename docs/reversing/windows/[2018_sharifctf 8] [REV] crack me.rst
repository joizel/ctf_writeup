=====================================================================
[2018_SharifCTF 8] [REV] Crack me!
=====================================================================

문제 내용
=====================================================================

Find the password.


동작 화면
=====================================================================

```sh
$ ./crackme
Enter Password:
aaaaaa
Try again
```

문제 풀이
=====================================================================

- 패스워드 값이 참이 아닐 경우, "Try again" 메시지 출력. 해당 출력이 발생되는 함수 부분을 디버깅을 통해 확인: sub_402140에서 해당 메시지가 출력됨
    - sub_402140이 받는 인자값 확인: v53
        - sub_401e70에서 v53값이 저장됨 (-1|0)
            - -1일 경우 "Try again"
            - 0일 경우 "Correct:Flag is MD5 Of Password"
                - 0이 출력되기 위해서는 Dst값이 whynxt과 일치해야함. Dst값은 입력값이 암호화된 값

