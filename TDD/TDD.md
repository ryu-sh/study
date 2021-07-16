# 자바 플레이그라운드 with Tdd, 클린코드 정리

## TDD란?
 - TDD = TFD(Test First Development) + 리팩토링
 - TDD란 프로그래밍 의사결정과 피드백 사이의 간극을 의식하고 이를 제어하는 기술이다. - 켄트벡, Test Driven Development by Example 중
 - TDD의 아이러니 중 하나는 테스트 기술이 아니라는 점이다. TDD는 분석 기술이며, 설계 기술이기도 하다. - 켄트벡, Test Driven Development by Example 중

## TDD를 하는 이유
 - 디버깅 시간을 줄여준다.
 - 동작하는 문서 역할을 한다.
 - 변화에 대한 두려움을 줄여준다.

![](https://articles.tbscg.com/wp-content/uploads/2015/11/tdd-cycle.png)

 - 실패하는 테스트를 구현한다.
 - 테스트가 성공하도록 프로덕션 코드를 구현한다.
 - 프로덕션 코드와 테스트 코드를 리팩토링한다.

## TDD 원칙
 - 원칙 1 - 실패하는 단위 테스트를 작성할 때까지 프로덕션 코드(production code)를 작성하지 않는다.
 - 원칙 2 - 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
 - 원칙 3 - 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.
