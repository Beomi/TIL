#2016.09.07(Wen)

## UnitTest
```
import unittest
```
위의 한줄로 시작하는 한줄 코드로 시작한다.
외부 모듈을 import해서 사용하는 때에 있어서 unittest.TestCase를 상속받은 class를 하나 생성 한 후,
그 클래스 내부에 테스트 함수를 만든다.
assertTrue, assertFalse, assertEqual등을 사용해 제대로 된 출력값이 되었는지 확인할 수 있다.

###[KoreaBankChecker#b4a5be2](https://github.com/Beomi/KoreaBankChecker/commit/b4a5be2808ffe2497770ba5beb26df0ff096a352)
이번 프로젝트는 목표가 단순했다. 이름과 값을 집어넣으면 그 거래가 실제로 알어났는지 유무를 확인하는 것.
이름과 금액을 입력하면 존재하는지를 오늘어제의 거래 내역을 체킹하고 이름(입금자/송금시 적는 이름)과 금액을 처리할 수 있다.
