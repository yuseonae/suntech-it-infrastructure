# 08. Monitoring and Log Design

## 1. Purpose

SunTech IT Infrastructure에서 운영되는 서버와 주요 서비스를 지속적으로 확인하고 장애 및 이상 징후를 신속하게 파악하기 위해 Monitoring Server와 Log Server를 구축한다.

이번 프로젝트에서는 Monitoring과 Log Management를 별도의 서버로 분리하여 구성한다.

- MON01: 서버 및 서비스 상태 모니터링
- LOG01: 주요 서버의 로그 중앙 수집 및 관리

이를 통해 다음과 같은 운영 환경을 구현한다.

    Server / Service
          |
          +--------→ MON01
          |            |
          |            ↓
          |       Status Monitoring
          |
          +--------→ LOG01
                       |
                       ↓
                  Log Collection
                       |
                       ↓
                  Event Analysis


## 2. Server Architecture

### MON01

Hostname:

`MON01`

IP Address:

`10.10.20.40`

Role:

- Monitoring Server
- Server Availability Monitoring
- Resource Monitoring
- Service Monitoring
- Alert Management


### LOG01

Hostname:

`LOG01`

IP Address:

`10.10.20.50`

Role:

- Central Log Server
- Log Collection
- Log Storage
- Security Event Analysis


## 3. Monitoring Target

MON01은 다음 서버와 서비스를 모니터링한다.

| Server | IP | Main Monitoring Target |
|---|---|---|
| DC01 | 10.10.20.10 | AD / DNS / Server |
| DC02 | 10.10.20.11 | AD / DNS / Server |
| DHCP01 | 10.10.20.12 | DHCP / Server |
| FILE01 | 10.10.20.20 | SMB / FSRM / Server |
| BACKUP01 | 10.10.20.30 | Backup / Storage |
| MON01 | 10.10.20.40 | Monitoring |
| LOG01 | 10.10.20.50 | Log Collection |
| WEB01 | 10.10.20.60 | IIS / Web |
| DB01 | 10.10.20.70 | PostgreSQL / Database |
| MAIL01 | 10.10.20.80 | Postfix / Dovecot / Mail |


## 4. Monitoring Categories

모니터링 항목은 다음과 같이 구분한다.

### Availability

서버 또는 서비스가 정상적으로 동작하고 있는지 확인한다.

- Ping
- TCP Port
- Service Status


### Resource

서버 자원 사용량을 확인한다.

- CPU
- Memory
- Disk
- Network


### Application / Service

주요 서비스의 정상 동작 여부를 확인한다.

- Active Directory
- DNS
- DHCP
- SMB
- IIS
- PostgreSQL
- Postfix
- Dovecot
- Backup


## 5. Monitoring Software

MON01은 오픈소스 Monitoring Solution을 사용한다.

우선 다음 솔루션을 사용한다.

`Zabbix`

Zabbix를 선택한 이유:

- 서버 모니터링에 적합
- Agent 기반 모니터링 지원
- SNMP 지원
- 서비스 및 Port 모니터링 가능
- Dashboard 제공
- Alert 기능 제공
- 기업 IT 인프라 실습에 적합


## 6. Zabbix Architecture

기본 구조:

    Zabbix Server
          |
          ↓
        MON01
          |
    +-----+-----+-----+-----+
    |     |     |     |     |
   DC01  FILE01 WEB01 DB01 MAIL01

각 서버의 상태와 자원 정보를 MON01에서 수집한다.


## 7. Monitoring Agent

Windows Server에는 Zabbix Agent를 설치하여 서버 상태 정보를 수집한다.

대상:

- DC01
- DC02
- DHCP01
- FILE01
- BACKUP01
- LOG01
- WEB01
- DB01

Linux Server인 MAIL01에도 Zabbix Agent를 설치한다.

수집 정보:

- CPU
- Memory
- Disk
- Network
- Process
- Service


## 8. Availability Monitoring

각 서버의 기본적인 가용성을 확인한다.

예:

    DC01
       ↓
    Ping
       ↓
    Response
       ↓
    UP

서버가 응답하지 않는 경우 MON01에서 장애 이벤트를 발생시킨다.


## 9. Port Monitoring

주요 서비스 Port를 모니터링한다.

| Server | Service | Port |
|---|---|---:|
| DC01 | DNS | 53 |
| DC02 | DNS | 53 |
| DHCP01 | DHCP | 67/68 |
| FILE01 | SMB | 445 |
| WEB01 | HTTP | 80 |
| WEB01 | HTTPS | 443 |
| DB01 | PostgreSQL | 5432 |
| MAIL01 | SMTP | 25 |
| MAIL01 | Submission | 587 |
| MAIL01 | IMAPS | 993 |

서비스 Port에 연결할 수 없는 경우 장애 여부를 확인한다.


## 10. Resource Monitoring

서버별 주요 자원을 모니터링한다.

### CPU

CPU 사용률이 지속적으로 높을 경우 성능 문제 가능성을 확인한다.


### Memory

Memory 사용량 및 Available Memory를 확인한다.


### Disk

Disk 사용량과 Free Space를 확인한다.

특히 다음 서버의 Disk 사용량을 중요하게 관리한다.

- FILE01
- BACKUP01
- DB01
- MAIL01
- LOG01

파일, 데이터베이스, 메일 및 로그 데이터가 지속적으로 증가할 수 있기 때문이다.

## 11. Service Monitoring

주요 Windows Service와 Linux Service의 상태를 확인한다.

### Windows

- Active Directory Domain Services
- DNS Server
- DHCP Server
- File Server
- IIS
- Zabbix Agent


### Linux

- Postfix
- Dovecot
- Zabbix Agent

Service가 중지되면 MON01에서 상태 변화를 확인할 수 있도록 구성한다.


## 12. Alert Design

Monitoring Server는 문제가 발생했을 때 Alert를 생성한다.

주요 Alert:

- Server Down
- High CPU Usage
- High Memory Usage
- Low Disk Space
- Service Down
- Port Down
- Network Connectivity Failure


## 13. Alert Severity

Alert의 중요도를 구분한다.

| Severity | Description |
|---|---|
| Information | 정보성 이벤트 |
| Warning | 주의가 필요한 상태 |
| Average | 일반적인 장애 |
| High | 높은 중요도의 장애 |
| Disaster | 심각한 서비스 장애 |

초기 구축에서는 Warning, High 수준의 Alert를 중심으로 구성한다.


## 14. Example Alert

예:

    Problem:
    FILE01 Disk Space Low

    Server:
    FILE01

    IP:
    10.10.20.20

    Status:
    Warning

    Action:
    Check Disk Usage


또 다른 예:

    Problem:
    WEB01 IIS Service Down

    Server:
    WEB01

    Service:
    IIS

    Status:
    High

    Action:
    Check IIS Service


## 15. Dashboard

MON01에는 전체 인프라 상태를 확인할 수 있는 Dashboard를 구성한다.

Dashboard에서 다음 정보를 확인한다.

- 전체 서버 상태
- 서버별 CPU
- 서버별 Memory
- Disk Usage
- Service Status
- Active Alerts
- Network Availability


## 16. Infrastructure Dashboard

예상 Dashboard 구조:

    SunTech Infrastructure

    Server Status
    --------------------------------
    DC01       UP
    DC02       UP
    DHCP01     UP
    FILE01     UP
    BACKUP01   UP
    WEB01      UP
    DB01       UP
    MAIL01     UP
    LOG01      UP
    --------------------------------

    Active Problems
    --------------------------------
    No Critical Problems


## 17. Log Server Architecture

LOG01은 여러 서버에서 발생하는 주요 로그를 중앙에서 수집한다.

기본 구조:

    DC01 --------\
    DC02 ---------\
    DHCP01 --------\
    FILE01 ---------\
    WEB01 ----------→ LOG01
    DB01 -----------/
    MAIL01 ---------/
    BACKUP01 -------/


## 18. Log Collection Software

LOG01에서는 오픈소스 Log Management Solution을 사용한다.

우선 다음 구성을 검토한다.

`Wazuh`

Wazuh를 선택하는 이유:

- 중앙 로그 수집 가능
- Windows / Linux 지원
- 보안 이벤트 분석 가능
- File Integrity Monitoring 지원
- Dashboard 제공
- 기업 보안 운영 환경 실습에 적합


## 19. Log Collection Targets

주요 로그 수집 대상:

### DC01 / DC02

- Windows Security Log
- System Log
- Application Log
- Active Directory 관련 이벤트
- DNS 관련 이벤트


### DHCP01

- DHCP Event Log
- Windows Security Log
- System Log


### FILE01

- File Access Audit
- Windows Security Log
- System Log
- FSRM 관련 이벤트


### WEB01

- IIS Access Log
- IIS Error Log
- Windows Event Log
- Application Log


### DB01

- PostgreSQL Log
- Windows Event Log
- Application Log


### MAIL01

- Postfix Log
- Dovecot Log
- Authentication Log
- System Log


### BACKUP01

- Windows Event Log
- Backup Event
- System Log


## 20. Windows Event Log

Windows Server에서 주요 Event Log를 수집한다.

기본 대상:

- Security
- System
- Application

특히 Security Log를 통해 인증 및 권한 관련 이벤트를 확인한다.


## 21. Active Directory Security Events

AD 환경에서는 다음과 같은 이벤트를 주요 모니터링 대상으로 삼는다.

- 사용자 로그인 성공
- 사용자 로그인 실패
- 계정 잠금
- 사용자 계정 변경
- 그룹 멤버십 변경
- 권한 변경

이를 통해 비정상적인 인증 또는 권한 변경 여부를 확인할 수 있도록 한다.

## 22. File Server Security Events

FILE01에서는 파일 접근 및 변경 이벤트를 중요하게 관리한다.

대상:

- 파일 접근
- 파일 생성
- 파일 수정
- 파일 삭제
- 권한 변경

예:

    User:
    SUN0001

    Resource:
    \\FILE01\Production\production-plan.xlsx

    Action:
    Modify

    Result:
    Success

이러한 이벤트를 LOG01에서 확인할 수 있도록 구성한다.


## 23. Web Server Log

WEB01의 IIS Log를 중앙에서 관리한다.

주요 정보:

- Client IP
- Request Time
- Requested URL
- HTTP Method
- Status Code
- User Agent

예:

    Client:
    10.10.10.101

    Request:
    GET /dashboard

    Status:
    200

웹 서비스 장애 및 비정상적인 접근 여부를 분석할 수 있도록 한다.


## 24. Database Log

DB01의 PostgreSQL Log를 중앙에서 관리한다.

주요 대상:

- Database Connection
- Authentication Failure
- Query Error
- Database Error
- Service Start / Stop

이를 통해 웹 애플리케이션에서 발생하는 데이터베이스 연결 문제를 추적할 수 있다.


## 25. Mail Server Log

MAIL01의 Mail Log를 중앙에서 관리한다.

주요 대상:

- SMTP Connection
- Authentication
- Mail Delivery
- Mail Rejection
- Relay Attempt
- IMAP Connection
- Authentication Failure

이를 통해 메일 송수신 장애 및 비정상적인 인증 시도를 분석한다.


## 26. Log Retention

로그는 무제한으로 저장하지 않고 보관 기간을 설정한다.

초기 프로젝트에서는 다음 기준을 사용한다.

- 일반 System Log: 30일
- Application Log: 30일
- Security Log: 90일
- 중요 보안 이벤트: 별도 보관 검토

실제 운영 환경에서는 기업의 보안 정책과 저장 공간을 고려하여 조정한다.


## 27. Log Security

LOG01에 수집된 로그는 임의로 변경되지 않도록 접근 권한을 제한한다.

원칙:

1. 일반 사용자 접근 제한
2. Log Administrator 권한 분리
3. Log Server 관리 계정 최소화
4. 로그 저장 공간 보호
5. 중요 로그 백업


## 28. Monitoring and Log Integration

MON01과 LOG01은 서로 다른 목적을 가지고 있지만 장애 대응 과정에서 함께 활용한다.

예:

    MON01
      |
      | Detect
      ↓
    WEB01 IIS Down
      |
      ↓
    Administrator
      |
      ↓
    LOG01
      |
      | Analyze
      ↓
    IIS Error Log
      |
      ↓
    Root Cause


## 29. Failure Response Scenario

### Scenario 1: Web Server Down

    MON01
      ↓
    WEB01 Down Alert
      ↓
    Administrator 확인
      ↓
    LOG01 확인
      ↓
    IIS / Windows Log 분석
      ↓
    원인 파악
      ↓
    Service Recovery
      ↓
    MON01 정상 상태 확인


### Scenario 2: File Access Issue

    User
      ↓
    File Access Failure
      ↓
    FILE01 Permission 확인
      ↓
    LOG01 Security Log 확인
      ↓
    User / Group / Permission 분석
      ↓
    권한 수정
      ↓
    Access Test


### Scenario 3: Database Service Failure

    MON01
      ↓
    DB01 PostgreSQL Down
      ↓
    LOG01 PostgreSQL Log 확인
      ↓
    Service / Storage / Connection 확인
      ↓
    PostgreSQL Recovery
      ↓
    Web Application Test


## 30. Backup

MON01과 LOG01의 설정 및 중요 데이터를 BACKUP01에 백업한다.

백업 대상:

### MON01

- Zabbix Configuration
- Monitoring Templates
- Dashboard Configuration


### LOG01

- Wazuh Configuration
- Important Logs
- Security Configuration


기본 구조:

    MON01 --------\
                   \
                    → BACKUP01
                   /
    LOG01 --------/


## 31. Implementation Scope

이번 프로젝트에서는 다음 항목을 실제 구축한다.

### Monitoring

- MON01
- Zabbix Server
- Zabbix Agent
- Server Availability Monitoring
- CPU Monitoring
- Memory Monitoring
- Disk Monitoring
- Port Monitoring
- Service Monitoring
- Dashboard
- Alert


### Logging

- LOG01
- Wazuh
- Windows Event Log Collection
- Linux Log Collection
- IIS Log Collection
- PostgreSQL Log Collection
- Mail Log Collection
- Security Event Analysis


## 32. Implementation Order

Monitoring / Log 환경은 다음 순서로 구축한다.

1. MON01 VM 생성
2. Linux Server 설치
3. Static IP 설정
4. DNS 설정
5. Zabbix Server 설치
6. Zabbix Agent 설치
7. Windows Server Agent 연결
8. MAIL01 Agent 연결
9. Server Availability Monitoring
10. Resource Monitoring
11. Service Monitoring
12. Port Monitoring
13. Dashboard 구성
14. Alert 구성
15. LOG01 VM 생성
16. Linux Server 설치
17. Wazuh 설치
18. Windows Agent 설치
19. Linux Agent 설치
20. Windows Event Log 수집
21. IIS Log 수집
22. PostgreSQL Log 수집
23. Mail Log 수집
24. Security Event 확인
25. Backup 구성
26. 장애 시나리오 테스트


## 33. Validation Checklist

### Monitoring

- MON01 정상 동작
- Zabbix Server 정상
- Agent 연결 정상
- 서버 UP/DOWN 확인
- CPU 확인
- Memory 확인
- Disk 확인
- Port 확인
- Service 상태 확인
- Alert 발생 확인
- Dashboard 정상


### Logging

- LOG01 정상 동작
- Wazuh 정상 동작
- Windows Log 수집
- Linux Log 수집
- IIS Log 수집
- PostgreSQL Log 수집
- Mail Log 수집
- Security Event 확인


### Incident Response

- Server Down Test
- Service Down Test
- Disk Usage Alert Test
- Authentication Failure Test
- File Access Event Test
- Web Error Test
- Database Error Test
- Mail Authentication Failure Test


## 34. Final Design

SunTech의 Monitoring / Log 환경은 다음과 같이 구성한다.

    +-------------------------------+
    |        SunTech Network        |
    +-------------------------------+
          |       |       |
          |       |       |
        Servers  WEB     MAIL
          |       |       |
          +-------+-------+
                  |
          +-------+-------+
          |               |
        MON01           LOG01
       Zabbix           Wazuh
          |               |
          ↓               ↓
     Monitoring       Log Analysis
          |               |
          +-------+-------+
                  |
               BACKUP01


MON01은 서버 및 서비스의 현재 상태를 확인하고 장애 발생 시 Alert를 생성한다.

LOG01은 각 서버에서 발생하는 Windows Event Log, Linux Log, IIS Log, Database Log 및 Mail Log를 중앙에서 수집한다.

장애가 발생하면 MON01을 통해 장애를 감지하고 LOG01의 로그를 통해 원인을 분석하는 방식으로 운영한다.

이를 통해 다음과 같은 IT 인프라 운영 프로세스를 구현한다.

    Monitor
       ↓
    Detect
       ↓
    Analyze
       ↓
    Troubleshoot
       ↓
    Recover
       ↓
    Verify

최종적으로 SunTech IT Infrastructure는 단순히 서버를 구축하는 것을 넘어 모니터링, 로그 분석, 장애 대응 및 복구까지 수행할 수 있는 운영 환경을 목표로 한다.
