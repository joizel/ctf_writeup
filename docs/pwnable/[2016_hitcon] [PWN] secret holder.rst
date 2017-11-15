============================================================================================================
[2016_hitcon] [PWN] Secret Holder
============================================================================================================

문제 내용
============================================================================================================

Break the Secret Holder and find the secret.
nc 52.68.31.117 5566
SecretHolder




============================================================================================================

프로그램 실행 시 3가지 선택 메뉴와 함께 다음과 같은 화면이 나온다.

.. code-block:: console

    Hey! Do you have any secret?
    I can help you to hold your secrets, and no one will be able to see it :)
    1. Keep secret
    2. Wipe secret
    3. Renew secret

1. Keep secret을 선택할 경우 다음과 같은 화면이 나온다.


.. code-block:: console

    Which level of secret do you want to keep?
    1. Small secret
    2. Big secret
    3. Huge secret


2. wipe secret을 선택할 경우 다음과 같은 화면이 나온다.

.. code-block:: console

    Which Secret do you want to wipe?
    1. Small secret
    2. Big secret
    3. Huge secret

3. Renew secret을 선택할 경우 다음과 같은 화면이 나온다.

.. code-block:: console

    Which Secret do you want to renew?
    1. Small secret
    2. Big secret
    3. Huge secret
