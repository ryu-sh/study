## 도메인 모델 시작
### 도메인 모델
 - 도메인은 객체와 비슷하다면 비슷하다고 할 수 있을 것 같다. 객체보단 큰 개념이라고 생각된다.
 - 도메인을 추출하고 모델링 한 뒤 프로그램을 설계하는 방향으로 가야한다.
 - 처음부터 잘 할수는 없다. 에자일 생각하듯이 설계하고 모델링 수정하고 설계 수정하고 이런식으로 반복이 된다.
### 엔티티와 밸류
 - 엔티티는 데이터베이스의 테이블 이라고 생각하면 지금까지 데이터 중심의 개발을 한것.
 - 엔티티와 데이터베이스 테이블 데이터는 다를 수 있다.
 - 밸류라는 것은 JPA의 값객체 개념으로 생각된다.
 - 엔티티와 데이터베이스 테이블 데이터가 다를 수 있다는게 밸류가 엔티티가 될 수도 있고(뒤 챕터에 설명됨) 테이블과 엔티티 매핑 설정에 따라 테이블이 될 수 있지만 다를수도 있다..
 - 엔티티 가장 큰 특징 : 식별자를 갖고 있다. (테이블의 pk와 다른 개념이다.)
 - 엔티티와 테이블의 차이 좀더 생각하고 뒤 챕터에서 정리해야겠다.
### 도메인 용어
 - 개발자 뿐만 아니라 기획자, PM 등등 프로젝트와 관련된 모든 사람들이 도메인에 사용된 용어, 도메인 이름등을 활용하여 커뮤니케이션을 해야 커뮤니케이션이 원할해진다.
 - 도메인 용어를 코드에 녹여내야 코드를 읽는 사람도 훨씬 편하게 읽을 수 있고 설명도 잘 할 수 있다.