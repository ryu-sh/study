# ThreadDump

## 파일 생성
 - jstack [pid] > dump.txt

## 무료 분석 사이트
 - https://fastthread.io/ft-index.jsp

## visualVm 사용하여 모니터링
1. rmiregistry 실행하기
 - $JAVA_HOME/bin 으로 이동
 - ./rmiregistry (port)&
3. policy 파일 만들기
```cat jstatd.all.policy 
grant codebase "file:${java.home}/../lib/tools.jar"{
        permission java.security.AllPermission;
};
```
 - jdk11에선 tools.jar가 존재하지 않아 실패..(검색을 해도 나오지 않는다..)
5. jstatd 실행하기
 - ./jstatd -J-Djava.security.policy=jstatd.all.policy -p (port)

### to-do
 - 분석 하는 방법 공부
