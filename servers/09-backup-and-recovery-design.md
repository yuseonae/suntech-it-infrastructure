# 09. Backup and Recovery Design

## 1. Purpose

SunTech IT Infrastructure에서 서버 장애, 데이터 손상, 사용자 실수 및 시스템 오류가 발생하더라도 주요 서비스와 데이터를 복구할 수 있도록 Backup and Recovery 환경을 구성한다.

이번 프로젝트에서는 별도의 Backup Server인 BACKUP01을 구축하고 주요 서버의 시스템 및 데이터를 백업한다.

단순히 백업을 수행하는 것에 그치지 않고 실제 데이터를 삭제하거나 서버 장애를 가정한 후 백업 데이터를 이용하여 복구하는 테스트까지 수행한다.

최종 목표는 다음과 같다.

Backup
→ Storage
→ Failure
→ Recovery
→ Validation


## 2. Backup Server

### BACKUP01

- Hostname: BACKUP01
- IP Address: 10.10.20.30
- Subnet Mask: 255.255.255.0
- Gateway: 10.10.20.1
- Primary DNS: 10.10.20.10
- Secondary DNS: 10.10.20.11

Role:

- Backup Server
- Backup Storage
- Recovery Data Storage


## 3. Backup Architecture

BACKUP01은 주요 서버의 백업 데이터를 중앙에서 관리한다.

Backup 대상:

- DC01
- DC02
- DHCP01
- FILE01
- WEB01
- DB01
- MAIL01
- MON01
- LOG01

기본 구조:

각 서버
→ BACKUP01
→ Backup Storage


## 4. Backup Scope

| Server | Main Backup Target |
|---|---|
| DC01 | System State / AD |
| DC02 | System State / AD |
| DHCP01 | DHCP Configuration |
| FILE01 | Department Files |
| WEB01 | Web Application / IIS Configuration |
| DB01 | PostgreSQL Database |
| MAIL01 | Mailbox / Mail Configuration |
| MON01 | Monitoring Configuration |
| LOG01 | Important Configuration / Logs |


## 5. Backup Principle

백업은 다음 원칙을 적용한다.

### 1. 중요 데이터 분리

업무 데이터와 시스템 설정을 구분하여 백업한다.

### 2. 원본과 백업 분리

원본 서버와 동일한 디스크에만 백업하지 않는다.

Source Server
→ BACKUP01

### 3. 복구 테스트

백업 파일이 존재하는 것만으로 복구 가능성을 판단하지 않는다.

실제 Restore Test를 수행하여 백업 데이터의 복구 가능성을 검증한다.


## 6. Backup Method

서버의 역할에 따라 적절한 백업 방식을 사용한다.

### Windows Server

Windows Server Backup을 사용한다.

주요 대상:

- DC01
- DC02
- DHCP01
- FILE01
- WEB01

### PostgreSQL

DB01은 PostgreSQL 자체 백업 기능을 사용한다.

- pg_dump

### Mail

MAIL01은 Mailbox와 Postfix / Dovecot 설정을 별도로 백업한다.

### Monitoring / Logging

MON01과 LOG01은 서비스 설정 및 필요한 데이터를 백업한다.


## 7. Backup Schedule

초기 프로젝트에서는 다음 기준을 사용한다.

| Target | Schedule |
|---|---|
| FILE01 | Daily |
| DB01 | Daily |
| WEB01 | Daily |
| MAIL01 | Daily |
| LOG01 | Daily |
| DC01 | Weekly |
| DC02 | Weekly |
| DHCP01 | Weekly |
| MON01 | Weekly |

중요 데이터는 일 단위 백업을 우선 적용한다.


## 8. Backup Retention

초기 실습 환경에서는 다음 기준을 사용한다.

- Daily Backup: 7일
- Weekly Backup: 4주

실제 운영 환경에서는 데이터 중요도, 저장 공간 및 기업 보안 정책에 따라 보존 기간을 조정한다.


## 9. Active Directory Backup

DC01과 DC02는 System State Backup을 수행한다.

주요 백업 대상:

- Active Directory Database
- SYSVOL
- Registry
- Boot Files
- System Configuration

AD 장애 발생 시 System State Backup을 이용하여 복구할 수 있도록 한다.


## 10. DHCP Backup

DHCP01의 DHCP 설정을 백업한다.

주요 대상:

- DHCP Scope
- Reservation
- DHCP Options
- DHCP Configuration

DHCP 서버 장애 발생 시 기존 IP 할당 환경을 복구할 수 있도록 한다.


## 11. File Server Backup

FILE01의 부서별 업무 파일을 백업한다.

공유 구조:

- Production
- Quality
- IT
- Sales

주요 대상:

- Excel
- Word
- PDF
- 업무 문서


## 12. File Restore Test

파일 서버에서는 사용자 실수로 파일이 삭제되는 상황을 가정한다.

Test Process:

1. 테스트 파일 생성
2. Backup 수행
3. 테스트 파일 삭제
4. BACKUP01에서 파일 복구
5. 파일 내용 확인
6. NTFS Permission 확인
7. 사용자 접근 테스트

복구된 파일의 데이터와 권한이 정상적으로 유지되는지 검증한다.


## 13. Web Server Backup

WEB01의 Web Application과 IIS 설정을 백업한다.

주요 대상:

- Web Application Files
- IIS Configuration
- Application Configuration
- HTTPS Certificate
- Web Service Configuration

Database는 DB01에서 별도로 백업한다.


## 14. Web Server Recovery

WEB01 장애를 가정한다.

Recovery Process:

1. WEB01 장애 확인
2. 서버 복구 또는 신규 서버 준비
3. IIS 설치
4. IIS Configuration Restore
5. Application Restore
6. HTTPS Configuration Restore
7. DNS 확인
8. DB Connection 확인
9. Web Service Test


## 15. Database Backup

DB01의 PostgreSQL Database를 별도로 백업한다.

Database:

suntech_portal

Backup Tool:

pg_dump

예상 Backup Format:

suntech_portal_backup.sql

또는

suntech_portal_backup.dump


Database Backup은 Windows Server Backup과 별도로 수행한다.


## 16. Database Restore Test

Database 장애 또는 데이터 손상을 가정한다.

Test Process:

1. 테스트 데이터 생성
2. PostgreSQL Backup 수행
3. 테스트 데이터 삭제
4. Database Restore
5. Table 확인
6. 데이터 확인
7. WEB01에서 데이터 조회

최종적으로 Web Application에서 복구된 데이터를 정상적으로 조회할 수 있는지 확인한다.


## 17. Mail Server Backup

MAIL01의 사용자 Mailbox와 Mail Service Configuration을 백업한다.

주요 대상:

- Mailbox Data
- Postfix Configuration
- Dovecot Configuration
- TLS Configuration
- User Account Configuration


## 18. Mail Server Recovery

MAIL01 장애를 가정한다.

Recovery Process:

1. Linux Server 복구
2. Postfix 설치
3. Postfix Configuration Restore
4. Dovecot 설치
5. Dovecot Configuration Restore
6. Mailbox Restore
7. TLS Configuration Restore
8. SMTP Test
9. IMAP Test


## 19. Monitoring Server Backup

MON01의 Monitoring Configuration을 백업한다.

주요 대상:

- Zabbix Configuration
- Monitoring Templates
- Dashboard Configuration
- Important Scripts
- Monitoring Database


## 20. Log Server Backup

LOG01의 주요 설정과 필요한 로그를 백업한다.

주요 대상:

- Wazuh Configuration
- Agent Configuration
- Important Security Events
- Important Logs

모든 로그를 영구 보관하지 않고 중요 이벤트와 설정을 중심으로 관리한다.


## 21. Backup Storage Structure

BACKUP01에서는 서버별 Backup Directory를 분리한다.

BACKUP01

- DC01
- DC02
- DHCP01
- FILE01
- WEB01
- DB01
- MAIL01
- MON01
- LOG01

서버별로 백업 데이터를 구분하여 관리한다.


## 22. Backup Access Control

백업 데이터에는 시스템 및 업무 데이터가 포함될 수 있으므로 접근 권한을 제한한다.

원칙:

- Backup Administrator만 관리
- 일반 사용자 접근 차단
- Backup Share Permission 제한
- NTFS Permission 적용
- Backup Account 권한 최소화


## 23. Backup Security

백업 데이터 자체의 보안을 확보한다.

적용 항목:

1. 일반 사용자 접근 차단
2. 관리자 계정 분리
3. 방화벽 적용
4. 불필요한 서비스 차단
5. 백업 데이터 접근 권한 제한
6. 중요 백업 별도 보호
7. 정기적인 Restore Test


## 24. Backup Monitoring

BACKUP01 자체도 MON01에서 모니터링한다.

Monitoring Target:

- Server Availability
- Disk Usage
- Backup Service
- Backup Job Status
- Backup Storage Capacity

특히 Backup Job 실패 여부를 확인한다.

백업이 실패했는데 이를 발견하지 못하면 실제 장애 발생 시 복구가 불가능할 수 있기 때문이다.


## 25. Recovery Priority

전체 인프라 장애 발생 시 다음 순서로 복구한다.

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

운영 및 보안 분석 환경 복구


## 26. Recovery Order

전체 인프라 장애를 가정할 경우 다음 순서로 복구한다.

1. DC01
2. DC02
3. DHCP01
4. FILE01
5. DB01
6. WEB01
7. MAIL01
8. MON01
9. LOG01


## 27. RPO

RPO(Recovery Point Objective)는 장애 발생 시 어느 시점까지 데이터를 복구할 것인지 나타내는 기준이다.

이번 프로젝트에서는 주요 업무 데이터에 대해 기본 RPO를 24시간 수준으로 설정한다.

예:

RPO = 24 hours

하루 단위 백업을 수행할 경우 최악의 상황에서 최대 약 24시간 전의 데이터까지 복구할 수 있다.


## 28. RTO

RTO(Recovery Time Objective)는 장애 발생 후 서비스를 어느 정도 시간 안에 복구할 것인지 나타내는 기준이다.

실습 환경에서는 기업 운영 수준의 고정된 RTO를 설정하기보다 실제 복구 테스트를 통해 복구 시간을 측정한다.

기록 항목:

- 장애 발생 시간
- 복구 시작 시간
- 서비스 복구 시간
- 정상 확인 시간


## 29. Recovery Test

백업 시스템의 신뢰성을 확인하기 위해 실제 Restore Test를 수행한다.

### Test 1: File Restore

1. FILE01에 테스트 파일 생성
2. Backup 수행
3. 테스트 파일 삭제
4. Backup에서 파일 복구
5. 파일 내용 확인
6. 권한 확인


### Test 2: Database Restore

1. DB01에 테스트 데이터 생성
2. Database Backup
3. 테스트 데이터 삭제
4. Database Restore
5. Table 확인
6. WEB01에서 데이터 조회


### Test 3: Web Recovery

1. WEB01 Backup
2. IIS 장애 상황 구성
3. IIS Configuration Restore
4. Application Restore
5. Web Service Test


### Test 4: Mail Recovery

1. 테스트 Mailbox 생성
2. Mail Backup
3. 테스트 Mailbox 삭제
4. Mailbox Restore
5. SMTP Test
6. IMAP Test


## 30. Recovery Validation

복구 후 다음 항목을 확인한다.

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


## 31. Disaster Recovery Scenario

전체 인프라 장애를 가정한다.

Infrastructure Failure
→ Server Recovery
→ DC01
→ DC02
→ DHCP01
→ FILE01
→ DB01
→ WEB01
→ MAIL01
→ MON01
→ LOG01
→ Service Validation


각 서버의 복구가 정상적으로 완료된 후 다음 서버로 넘어간다.


## 32. Backup Checklist

### Daily

- FILE01 Backup
- DB01 Backup
- WEB01 Backup
- MAIL01 Backup
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


## 33. Implementation Order

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
19. MON01에서 BACKUP01 모니터링
20. File Restore Test
21. Database Restore Test
22. Web Restore Test
23. Mail Restore Test
24. Recovery Time 측정


## 34. Final Architecture

전체 Backup Architecture:

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

- System Backup
- File Backup
- Database Backup
- Mail Backup
- Configuration Backup


MON01
→ Backup Server Monitoring


## 35. Final Recovery Process

SunTech의 Backup and Recovery Process는 다음과 같이 구성한다.

Backup
→ Verify
→ Monitor
→ Failure
→ Identify
→ Restore
→ Validate
→ Resume


백업 성공 여부를 확인하고 장애 발생 시 적절한 백업 데이터를 선택하여 복구한다.

복구 후에는 서버 상태, 서비스 상태, 데이터, 권한 및 네트워크 연결을 검증한다.


## 36. Final Design Principle

SunTech Backup Infrastructure는 단순한 데이터 복사 방식이 아니라 장애 발생을 전제로 한 Recovery 중심의 환경으로 설계한다.

각 서버의 역할에 따라 적절한 백업 방식을 적용하고 BACKUP01에서 백업 데이터를 중앙 관리한다.

Active Directory는 System State를 백업하고 FILE01은 업무 파일을 백업하며 DB01은 PostgreSQL Database를 별도로 백업한다.

WEB01과 MAIL01은 서비스 설정과 데이터를 함께 백업하고 MON01과 LOG01은 운영 및 보안 환경을 복구할 수 있도록 필요한 설정과 데이터를 백업한다.

또한 MON01을 통해 BACKUP01과 Backup Job의 상태를 확인하여 백업 실패를 조기에 발견한다.

최종적으로 실제 파일 삭제, 데이터 삭제 및 서비스 장애를 가정한 Restore Test를 수행하여 백업 데이터가 실제로 복구 가능한지 검증한다.

이를 통해 다음과 같은 운영 체계를 구현한다.

Prevent
→ Backup
→ Detect
→ Recover
→ Validate
→ Resume
