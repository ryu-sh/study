# 나쁜상황에 대비한 스프링 클라우드와 넷플릭스 히스트릭스의 클라이언트 회복성 패턴

서비스하나가 충돌할때 그것을 감지하기는 쉽지만 느려질때 성능 저하를 감지하고 우회하는 것이 어려운 이유
- 서비스 저하는 간헐적으로 발생 확산 가능
- 원격서비스 호출은 대개 동기식이며 오래 걸리는 호출을 중단하지 않음
- 애플리케이션은 대개 부분적 저하가 아닌 원격 자원의 완전한 장애를 처리하도록 설계됨 (서비스가 완전히 실패하지 않으나 기능이 저하된채로 동작하게 됨)

## 클라이언트 회복성 패턴이란?
원격 서비스가 에러를 던지거나 제대로 동작하지 못해 원격 자원의 접근 실패시, 원격자원을 호출하는 클라이언트의 충돌을 막는데 초점을 맞춘다. 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FovwAS%2FbtqF0ZncICN%2FD67UAnS3iWzO0Tp6Ktv190%2Fimg.png)

1. 클라이언트 측 부하분산 client-side load balancing
- 클라이언트가 넷플릭스 유레카 같은 서비스 디스커버리 에이전트를 이용해 서비스의 모든 인스턴스를 검색한 후 해당 서비스 인스턴스의 실제 위치를 캐싱하는 것.
- 서비스 소비자가 서비스 인스턴스를 호출 할 때마다 클라이언트 측 로드 밸런서는 서비스 위치 풀에서 관리하는 서비스 위치를 하나씩 전달함.
- 클라이언트 로드밸런서는 중간에서 서비스 인스턴스가 불량인지 에러를 전달하는지를 감지해서 문제가 되는 서비스 인스턴스를 제거하고 호출 전달을 막는다.
(4장에서 설명한 리본라이브러리를 사용한 클라이언트 측 부하분산과 동일한 일이다.)
 
2. 회로 차단기 circuit breaker
- 유입된 과전류를 차단하는 것처럼 전기 회로 차단기를 본떠 만든 회복성 패턴.
- 원격 서비스 호출을 모니터링 해 호출이 어느정도 실패하면 빨리 실패하게 만들어 더이상 호출되지 않도록 차단한다.
 
3. 폴백 fallback
- 사용자 호출에 문제가 있어도 예외를 표시하지않고 나중에 해당요청을 수행할 수 있게 전달.
- 사용자 요청을 큐에 입력하는 작업과 연관된다.
 
4. 벌크헤드 bulkhead
- 선체에 구멍이 뚫려도 구획으로 분리되어있어 침수 구역을 격리해 침몰을 예방하는 것처럼, 원격 자원에 대한 호출을 자원별 스레드 풀로 분리하여 어플리케이션이 다운되지 않도록 한다.
- 각 원격자원은 스레드 풀에 할당되고, 한 서비스 호출 스레드 풀이 포화되더라도 다른 서비스 호출은 포화되지 않는다

## 클라이언트 회복성이 중요한 이유
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcbIEp3%2FbtqF2StK1Nb%2Fb5PuTR83c8ovqGpZUVRTi1%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvSIsn%2FbtqFZ5BLFjS%2F3HKoDcEomLmT9ilqWCrQV1%2Fimg.png)
1. 첫번째 성공적인 기본 시나리오
 - 회로차단기가 타이머를 설정하고 그 전에 호출이 완료되면 정상적으로 작업 수행
2. 서비스 저하 시나리오. 폴백없는 회로차단기
- 서비스 B가 회로차단기를 이용해 서비스C를 호출하지만 타이머가 만료되기전 호출이 완료되지않아 커넥션 차단.
3. 서비스 저하에서 원활하게 회복하는 시나리오. 폴백있는 회로차단기
- 계속 실패한 호출을 빠르게 실패시키고 대체 코드 폴백을 사용하여 다른곳의 데이터를 조회하게 만들수 있다. 
- 서비스 C 호출을 막는동안 서비스 C는 복구할 여유가 생기며 성능 저하를 회복하고, 회로차단기가 다시 간헐적으로 호출을 시도해 성공하면 회로차단기를 재설정하게 된다.
## 스프링 클라우드와 히스트릭스를 위한 라이선싱 서버 설정
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```
Main Class에 @EnableCircuitBreaker 어노테이셔 추가

## 히스트릭스를 사용한 회로 차단기 구현
- 스프링 클라우드와 히스트릭스 애너테이션을 사용해 회로차단기 패턴으로 원격호출을 감싼다.
- 라이선싱 및 조직 서비스 모두 자기 데이터베이스에 대한 호출을 히스트릭스 회로 차단기에 연결
- 두 서비스 사이의 호출을 히스트릭스에 연결

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F6uAYY%2FbtqF0SoHDHK%2FFGttnTK861oUkeuSuI2Qh1%2Fimg.png)

@HystrixCommand 애너테이션을 사용해 히스트릭스 회로차단기가 관리하는 자바 클래스 메서드라고 표시한다.
스프링 프레임워크가 메서드를 감싸는 프록시를 동적으로 생성하고 원격 호출을 처리하기 위해 확보한
스레드가 있는 스레드 풀로 해당 메서드에 대한 모든 호출을 관리한다.

```java
@HystrixCommand(
  commandProperties={ @HystrixProperty(
  name="execution.isolation.thread.timeoutInMilliseconds", value="12000")
  })
public List<License> getLicensesByOrg(String organizationId){
  randomlyRunLong();
  return licenseRepository.findByOrganizationId(organizationId);
}
```

## 폴백 프로세싱
- 회로차단기가 호출울 중단하거나 호출이 실패할 경우 폴백전략을 구현
- 중간자를 두어 서비스 실패를 가로채 다른대안을 선택할 기회를 줄 수 있다.
- 실행 폴백메서드는 애너테이션으로 보호하려는 메서드와 동일 클래스에 있어야 한다.

```java
@HystrixCommand(fallbackMethod = "buildFallbackLicenseList") //호출이 실패할때 불러오는 클래스함수를 정의
  public List<License> getLicensesByOrg(String organizationId){
    randomlyRunLong();
    return licenseRepository.findByOrganizationId(organizationId);
  }
private List<License> buildFallbackLicenseList(String organizationId){ 
  List<License> fallbackList = new ArrayList<>();
  License license = new License()
    .withId("0000000-00-00000")
    .withOrganizationId( organizationId )
    .withProductName("Sorry no licensing information currently available");
  fallbackList.add(license);
  return fallbackList;
}
```

## 벌크헤드 패턴 구현
- 서비스 내 개별 스레드풀을 사용해 서비스 호출을 격리하고, 호출되는 원격자원간에 벌크헤드 구축.
- 벌크헤드 패턴은 원격 자원 호출을 자신의 스레드 풀에 격리하기 때문에 컨테이너의 비정상 종료를 방지한다.
- 호출량이 많은 서비스가 히스트릭스의 기본 스레드 풀의 모든 스레드를 차지하게 되므로 문제가 생길수 있다. 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F2t76Q%2FbtqF16fj6im%2FKmvNpWVoNEhfPQa2Jmj5uK%2Fimg.png)

- maxQueueSize 속성에 대해 주의할것
-  값을 -1로 설정하면 유입된 호출을 유지하는데 SynchronousQueue 동기식 큐가 사용되고, 가용 스레드 수보다 더 많은 요청을 처리할수 있다. 1보다 큰 값으로 설정하면  LinkedBlockingQueue 사용해 더 많은 요청을 큐에 넣을수 있다.
- 스레드 풀이 처음 초기화될 때만 설정할 수 있음.
- 스레드 풀의 적정크기는? 
  - (서비스가 정상일때 최고점에서 초당 요청 수 x 99 백분위 수 지연 시간(초)) + 오버헤드를 대비한 소량의 추가 스레드
```java
@HystrixCommand(fallbackMethod = "buildFallbackLicenseList",
             threadPoolKey = "licenseByOrgThreadPool", //스레드풀 고유이름 설정
             threadPoolProperties = { // 스레드풀 동작을 정의하고 설정
                 @HystrixProperty(name = "coreSize",value="30"), //스레드 개수 정의
                 @HystrixProperty(name="maxQueueSize", value="10")}) //스레드풀 앞에 배치할 큐와 큐에 넣을 요청수 정의
public List<License> getLicensesByOrg(String organizationId){ 
  return licenseRepository.findByOrganizationId(organizationId);
}
```

## 히스트릭스 세부 설정
## 스레드 컨텍스트와 히스트릭스
