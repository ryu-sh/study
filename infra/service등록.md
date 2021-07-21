## service 파일 만들기

```
vi /etc/systemd/system/{servicename}.service
```

### 내용(에시)
```
[Unit]
Description=LALP-EMP-ID-ADMIN-SERVER
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c "exec sh /home/ldccblock/ldcc-emp-id/admin-server/start.sh"

User=ldccblock
Group=ldccblock

[Install]
WantedBy=multi-user.target
```

### 부가 설명
Unit
  - Description: 해당 유닛에 대한 설명
  - Requires : 상위 의존성 구성, 포함하는 유닛이 정상적이어야 실행
  - RequiresOverridable : 상위 의존성 구성이며 이것이 실패하더라도 무시하고 유닛을 시작
  ...

Service
  - Type: [simple | forking | oneshot | notify | dbus] 유닛의 타입
  - Environment: 해당 유닛에서 사용할 환경 변수 선언
  - ExecStart: 시작 명령을 정의
  - ExecStop: 중지 명령을 정의

Install
  - WantedBy: 유닛을 등록하기 위한 종속성 검사. 유닛을 등록할 때 등록에 필요한 유닛을 지정

### 재부팅 후에도 서비스가 실행되도록 등록
```
systemctl enable {servicename}
```
