# 09. Backup and Recovery Design

## 1. Purpose

SunTech IT Infrastructure에서 서버 장애, 데이터 손상, 사용자 실수 및 시스템 오류가 발생하더라도 주요 서비스와 데이터를 복구할 수 있도록 Backup and Recovery 환경을 구성한다.

이번 프로젝트에서는 별도의 Backup Server인 BACKUP01을 구축하고 주요 서버의 시스템 및 데이터를 백업한다.

단순히 백업을 수행하는 것에 그치지 않고 실제 데이터를 삭제하거나 서버 장애를 가정한 후 백업 데이터를 이용하여 복구하는 테스트까지 수행한다.

최종 목표는 다음과 같다.

    Backup
       ↓
    Storage
       ↓
    Failure
       ↓
    Recovery
       ↓
    Validation


## 2. Backup Server

### BACKUP01

Hostname:

`BACKUP01`

IP Address:

`10.10.20.30`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

Primary DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

Role:

- Backup Server
- Backup Storage
- Recovery Data Storage


## 3. Backup Network

BACKUP01은 Server Network에 배치한다.

Server Network:

`10.10.20.0/24`

Backup Server:

`10.10.20.30`

백업 대상 서버는 BACKUP01에 필요한 백업 데이터만 전송할 수 있도록 네트워크 및 방화벽 정책을 적용한다.


## 4. Backup Architecture

전체적인 백업 구조는 다음과 같다.

    DC01 --------\
    DC02 ---------\
    DHCP01 --------\
    FILE01 ---------\
    WEB01 ----------\
    DB01 ------------→ BACKUP01
    MAIL01 ----------/
    MON01 -----------/
    LOG01 -----------/


BACKUP01은 여러 서버의 백업 데이터를 중앙에서 관리한다.


## 5. Backup Scope

백업 대상은 서버의 역할에 따라 다르게 구성한다.

| Server | Backup Target |
|---|---|
| DC01 | System State / AD |
| DC02 | System State / AD |
| DHCP01 | DHCP Configuration |
| FILE01 | Department Files |
| WEB01 | Web Application / IIS Configuration |
| DB01 | Database |
| MAIL01 | Mailbox / Configuration |
| MON01 | Monitoring Configuration |
| LOG01 | Important Configuration / Logs |


## 6. Backup Principle

백업은 다음 세 가지 원칙을 기본으로 한다.

### 1. 중요한 데이터는 별도로 보호한다.

업무 데이터와 시스템 설정을 구분하여 백업한다.


### 2. 백업 데이터는 원본 서버와 분리한다.

원본 서버와 동일한 디스크 또는 동일한 위치에만 백업하지 않는다.

    Source Server
         |
         ↓
      BACKUP01


### 3. 복구 테스트를 수행한다.

백업 파일이 존재하는 것만으로는 실제 복구 가능성을 보장할 수 없다.

따라서 정기적으로 복구 테스트를 수행한다.


## 7. Backup Types

이번 프로젝트에서는 다음 백업 방식을 사용한다.

### Full Backup

전체 데이터를 백업한다.

장점:

- 복구가 단순함
- 백업 데이터 관리가 쉬움

단점:

- 저장 공간 사용량이 큼
- 백업 시간이 길어질 수 있음


### Incremental Backup

마지막 백업 이후 변경된 데이터만 백업한다.

장점:

- 저장 공간 절약
- 백업 시간 단축

단점:

- 복구 시 여러 백업 데이터가 필요할 수 있음


이번 프로젝트에서는 서버 역할에 따라 Full Backup 또는 변경 데이터 중심 백업을 적용한다.


## 8. Backup Schedule

초기 프로젝트에서는 다음 주기를 사용한다.

| Backup Target | Schedule |
|---|---|
| FILE01 | Daily |
| DB01 | Daily |
| MAIL01 | Daily |
| WEB01 | Daily |
| DC01 | Weekly |
| DC02 | Weekly |
| DHCP01 | Weekly |
| MON01 | Weekly |
| LOG01 | Daily |


중요 데이터는 일 단위 백업을 우선 적용한다.


## 9. Backup Retention

초기 환경에서는 다음 보존 정책을 적용한다.

- Daily Backup: 7일
- Weekly Backup: 4주
- 중요 백업: 별도 보관 검토

백업 저장 공간이 제한적인 실습 환경에서는 실제 저장 용량에 따라 보존 기간을 조정할 수 있다.

## 10. Windows Server Backup

Windows Server 기반 서버에는 Windows Server Backup을 우선 사용한다.

주요 대상:

- DC01
- DC02
- DHCP01
- FILE01
- WEB01
- BACKUP01
- MON01
- LOG01

Windows Server Backup을 이용하여 시스템 상태 및 필요한 서버 데이터를 백업한다.


## 11. Active Directory Backup

Active Directory Domain Controller는 System State Backup을 수행한다.

대상:

- DC01
- DC02

주요 백업 대상:

- Active Directory Database
- SYSVOL
- Registry
- Boot Files
- System Configuration

Active Directory 장애 발생 시 System State Backup을 이용하여 복구할 수 있도록 한다.


## 12. DHCP Backup

DHCP01의 DHCP 설정과 Scope 정보를 백업한다.

백업 대상:

- DHCP Scope
- Reservation
- DHCP Options
- DHCP Configuration

DHCP 서버 장애 발생 시 설정을 복구하여 기존 IP 할당 환경을 유지할 수 있도록 한다.


## 13. File Server Backup

FILE01에서는 실제 업무 파일을 주요 백업 대상으로 지정한다.

예상 공유 구조:

    FILE01
       |
       +-- Production
       |
       +-- Quality
       |
       +-- IT
       |
       +-- Sales


각 부서의 업무 파일을 BACKUP01에 백업한다.

특히 다음과 같은 파일을 복구할 수 있도록 테스트한다.

- Excel
- Word
- PDF
- 업무 문서


## 14. File Server Recovery

파일 서버에서는 사용자 실수로 파일이 삭제되는 상황을 가정한다.

시나리오:

    User
      ↓
    File Delete
      ↓
    Backup 확인
      ↓
    File Restore
      ↓
    Permission 확인
      ↓
    User Access Test


복구 후 기존 파일의 내용과 NTFS / Share Permission이 정상적으로 적용되는지 확인한다.


## 15. Web Server Backup

WEB01에서는 Web Application과 IIS 설정을 백업한다.

주요 대상:

- Web Application Files
- IIS Configuration
- Application Configuration
- HTTPS Certificate
- Web Service Configuration

DB 데이터 자체는 DB01에서 별도로 백업한다.


## 16. Web Server Recovery

WEB01 장애를 가정하여 Web Application을 복구한다.

시나리오:

    WEB01 Failure
         ↓
    New / Repaired WEB01
         ↓
    Restore IIS Configuration
         ↓
    Restore Application
         ↓
    Check DNS
         ↓
    Check DB Connection
         ↓
    Web Service Test


복구 후 사용자가 내부 웹 서비스에 정상적으로 접근할 수 있는지 확인한다.


## 17. Database Backup

DB01의 PostgreSQL Database는 Database 수준의 별도 백업을 수행한다.

Database:

`suntech_portal`

백업 방식:

`pg_dump`

예상 백업 파일:

`suntech_portal_backup.sql`

또는 Custom Format을 사용할 경우:

`suntech_portal_backup.dump`


Database Backup은 Windows Server Backup과 별도로 수행한다.


## 18. Database Recovery

Database 장애 또는 데이터 손상을 가정하여 PostgreSQL Database를 복구한다.

시나리오:

    DB01
      ↓
    Data Loss
      ↓
    PostgreSQL Backup
      ↓
    Database Restore
      ↓
    Table Check
      ↓
    Application Connection Test


복구 후 다음 데이터를 확인한다.

- Users
- Notices
- Service Requests
- Equipment
- Request History


복구된 데이터가 정상적으로 조회되는지 WEB01에서 최종 확인한다.

## 19. Mail Server Backup

MAIL01의 사용자 Mailbox와 Mail Service 설정을 백업한다.

주요 대상:

- Mailbox Data
- Postfix Configuration
- Dovecot Configuration
- TLS Configuration
- User Account Configuration


## 20. Mail Server Recovery

MAIL01 장애를 가정하여 Mail Service를 복구한다.

시나리오:

    MAIL01 Failure
         ↓
    Linux Server Recovery
         ↓
    Postfix Configuration Restore
         ↓
    Dovecot Configuration Restore
         ↓
    Mailbox Restore
         ↓
    SMTP Test
         ↓
    IMAP Test


복구 후 기존 사용자의 Mailbox와 내부 메일 송수신 기능을 확인한다.


## 21. Monitoring Server Backup

MON01에서는 Monitoring 설정을 백업한다.

주요 대상:

- Zabbix Configuration
- Monitoring Templates
- Dashboard Configuration
- Important Scripts
- Database


MON01 장애 발생 시 Monitoring 환경을 다시 구성할 수 있도록 필요한 설정을 백업한다.


## 22. Log Server Backup

LOG01에서는 필요한 설정과 중요 로그를 백업한다.

주요 대상:

- Wazuh Configuration
- Agent Configuration
- Important Security Events
- Important Logs


모든 로그를 영구적으로 백업하지 않고 중요 보안 이벤트와 필요한 설정을 중심으로 관리한다.


## 23. Backup Storage Structure

BACKUP01에서는 서버별 백업 영역을 분리한다.

예상 구조:

    BACKUP01
       |
       +-- DC01
       |
       +-- DC02
       |
       +-- DHCP01
       |
       +-- FILE01
       |
       +-- WEB01
       |
       +-- DB01
       |
       +-- MAIL01
       |
       +-- MON01
       |
       +-- LOG01


각 서버의 백업 데이터를 구분하여 저장하고 관리한다.


## 24. Backup Access Control

백업 데이터에는 주요 시스템 및 업무 데이터가 포함될 수 있으므로 접근 권한을 제한한다.

원칙:

- Backup Administrator만 관리
- 일반 사용자 접근 차단
- 백업 저장소 Share Permission 제한
- NTFS Permission 적용
- Backup Account 권한 최소화


## 25. Backup Security

백업 데이터 자체가 공격받는 상황을 고려한다.

보안 원칙:

1. Backup Server 일반 사용자 접근 차단
2. 관리 계정 분리
3. 방화벽 적용
4. 불필요한 서비스 차단
5. 백업 데이터 접근 로그 관리
6. 중요 백업 별도 보호
7. 정기적인 Restore Test


## 26. Backup Failure Monitoring

BACKUP01 자체도 MON01에서 모니터링한다.

모니터링 대상:

- Backup Server Availability
- Disk Usage
- Backup Service
- Backup Job Status
- Backup Storage Capacity


백업 작업이 실패하더라도 이를 발견하지 못하면 실제 장애 발생 시 복구가 불가능할 수 있기 때문에 Backup Job의 성공 여부를 확인한다.


## 27. Recovery Priority

모든 서버를 동시에 복구할 필요는 없으므로 우선순위를 설정한다.

### Priority 1

- DC01
- DC02

인증 및 DNS 환경 복구


### Priority 2

- DHCP01
- FILE01

기본적인 내부 업무 환경 복구


### Priority 3

- DB01
- WEB01
- MAIL01

업무 서비스 복구


### Priority 4

- MON01
- LOG01

운영 및 분석 환경 복구


## 28. Recovery Order

전체 인프라 장애를 가정하는 경우 다음 순서로 복구한다.

1. DC01
2. DC02
3. DHCP01
4. FILE01
5. DB01
6. WEB01
7. MAIL01
8. MON01
9. LOG01

기본적인 인프라와 인증 환경을 먼저 복구한 후 업무 서비스를 복구한다.

## 29. RPO

RPO(Recovery Point Objective)는 장애 발생 시 어느 시점까지의 데이터를 복구할 것인지 나타내는 기준이다.

예:

`RPO = 24 hours`

하루 단위 백업을 수행할 경우 최악의 상황에서 최대 약 24시간 전의 데이터까지 복구할 수 있다.

이번 프로젝트에서는 주요 업무 데이터에 대해 기본 RPO를 24시간 수준으로 설정한다.


## 30. RTO

RTO(Recovery Time Objective)는 장애 발생 후 서비스를 어느 정도 시간 안에 복구할 것인지 나타내는 기준이다.

이번 프로젝트에서는 실습 환경의 특성을 고려하여 정확한 기업 운영 수준의 RTO를 목표로 하기보다 서버별 복구 절차와 실제 복구 시간을 측정하는 것을 목표로 한다.

복구 테스트 시 다음 시간을 기록한다.

- 장애 발생 시간
- 복구 시작 시간
- 서비스 복구 시간
- 정상 확인 시간


## 31. Recovery Test

백업 시스템의 신뢰성을 확인하기 위해 실제 Restore Test를 수행한다.

### Test 1: File Restore

1. FILE01의 테스트 파일 생성
2. Backup 수행
3. 테스트 파일 삭제
4. Backup에서 파일 복구
5. 파일 내용 확인
6. 사용자 접근 권한 확인


### Test 2: Database Restore

1. DB01에 테스트 데이터 생성
2. Database Backup 수행
3. 테스트 데이터 삭제
4. Database Restore
5. Table 확인
6. WEB01에서 데이터 조회


### Test 3: Web Service Recovery

1. WEB01 Web Application Backup
2. IIS Service 중지 또는 장애 상황 구성
3. Configuration Restore
4. Application Restore
5. DNS 확인
6. Web Service 접속 테스트


### Test 4: Mail Recovery

1. 테스트 Mailbox 생성
2. Mail Backup 수행
3. 테스트 Mailbox 삭제
4. Mailbox Restore
5. SMTP Test
6. IMAP Test


## 32. Recovery Validation

복구가 완료되었다고 판단하기 위해 단순히 파일 또는 서비스가 존재하는지만 확인하지 않는다.

다음 항목을 검증한다.

### Server

- Server Boot 정상
- Network 정상
- DNS 정상
- Domain 정상


### Application

- Service Running
- Port Open
- User Access 가능


### Data

- 데이터 존재
- 데이터 내용 정상
- 데이터 무결성 확인


### Permission

- 기존 사용자 접근 가능
- 접근 제한 대상 접근 불가


### Monitoring

- MON01에서 정상 상태 확인


## 33. Disaster Recovery Scenario

전체 인프라 장애를 가정한다.

예:

    Infrastructure Failure
             ↓
        Server Recovery
             ↓
           DC01
             ↓
           DC02
             ↓
          DHCP01
             ↓
          FILE01
             ↓
           DB01
             ↓
           WEB01
             ↓
          MAIL01
             ↓
          MON01
             ↓
          LOG01
             ↓
        Service Validation


각 서버 복구 후 다음 서버로 넘어간다.


## 34. Backup Checklist

### Daily

- FILE01 Backup
- DB01 Backup
- MAIL01 Backup
- WEB01 Backup
- LOG01 Backup
- Backup Job Status 확인


### Weekly

- DC01 System State Backup
- DC02 System State Backup
- DHCP01 Configuration Backup
- MON01 Configuration Backup
- Backup Storage 점검


### Monthly

- Restore Test
- Backup Integrity Check
- Storage Capacity Check
- Backup Policy Review


## 35. Implementation Order

Backup 환경은 다음 순서로 구축한다.

1. BACKUP01 VM 생성
2. Windows Server 설치
3. Static IP 설정
4. Domain Join
5. Backup Storage 구성
6. Windows Server Backup 설치
7. DC01 Backup 구성
8. DC02 Backup 구성
9. DHCP01 Backup 구성
10. FILE01 Backup 구성
11. WEB01 Backup 구성
12. DB01 PostgreSQL Backup 구성
13. MAIL01 Backup 구성
14. MON01 Backup 구성
15. LOG01 Backup 구성
16. Backup Schedule 설정
17. Backup Retention 설정
18. Backup Access Control 설정
19. MON01에서 Backup Server 모니터링
20. File Restore Test
21. Database Restore Test
22. Web Restore Test
23. Mail Restore Test
24. Recovery Time 측정


## 36. Final Architecture

SunTech Backup Architecture는 다음과 같이 구성한다.

    DC01 --------\
    DC02 ---------\
    DHCP01 --------\
    FILE01 ---------\
    WEB01 ----------\
    DB01 ------------→ BACKUP01
    MAIL01 ----------/
    MON01 -----------/
    LOG01 -----------/


    BACKUP01
        |
        +-- System Backup
        +-- File Backup
        +-- Database Backup
        +-- Mail Backup
        +-- Configuration Backup


    MON01
        |
        ↓
    Backup Monitoring


## 37. Final Recovery Process

SunTech의 백업 및 복구 프로세스는 다음과 같이 운영한다.

    Backup
       ↓
    Verify
       ↓
    Monitor
       ↓
    Failure
       ↓
    Identify
       ↓
    Restore
       ↓
    Validate
       ↓
    Service Recovery


백업 성공 여부를 확인하고, 장애 발생 시 적절한 백업 데이터를 선택하여 복구한다.

복구 후에는 서버 상태, 서비스 상태, 데이터, 권한 및 네트워크 연결을 검증한다.


## 38. Final Design Principle

SunTech Backup Infrastructure는 단순한 데이터 복사 방식이 아니라 장애 발생을 전제로 한 Recovery 중심의 환경으로 설계한다.

각 서버의 역할에 따라 적절한 백업 방식을 적용하고, BACKUP01에서 백업 데이터를 중앙 관리한다.

Active Directory는 System State를 백업하고, FILE01은 업무 파일을 백업하며, DB01은 PostgreSQL Database를 별도로 백업한다.

WEB01과 MAIL01은 서비스 설정과 데이터를 함께 백업하고 MON01과 LOG01은 운영 및 보안 환경을 복구할 수 있도록 필요한 설정과 데이터를 백업한다.

또한 MON01을 통해 Backup Server와 Backup Job의 상태를 확인하여 백업 실패를 조기에 발견한다.

최종적으로 실제 파일 삭제, 데이터 삭제 및 서비스 장애를 가정한 Restore Test를 수행하여 백업 데이터가 실제로 복구 가능한지 검증한다.

이를 통해 다음과 같은 운영 체계를 구현한다.

    Prevent
       ↓
    Backup
       ↓
    Detect
       ↓
    Recover
       ↓
    Validate
       ↓
    Resume
