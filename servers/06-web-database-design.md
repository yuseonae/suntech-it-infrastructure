# 06. Web and Database Server Design

## 1. Purpose

SunTech의 내부 업무 시스템을 운영하기 위한 Web Server와 Database Server를 구성한다.

Web Server는 사용자가 웹 브라우저를 통해 접근하는 사내 업무 서비스를 제공하고, Database Server는 해당 서비스에서 사용하는 업무 데이터를 저장한다.

이번 프로젝트에서는 실제 기업 환경을 가정하여 다음과 같은 구조를 구현한다.

    User PC
        |
        ↓
      WEB01
        |
        ↓
      DB01

이를 통해 네트워크, 서버, Active Directory, 데이터베이스 및 웹 서비스를 하나의 인프라 환경으로 연결한다.


## 2. Service Architecture

Web Server와 Database Server를 역할별로 분리한다.

### WEB01

Hostname:

`WEB01`

IP Address:

`10.10.20.60`

Role:

- Web Server
- Internal Web Application
- Application Service

### DB01

Hostname:

`DB01`

IP Address:

`10.10.20.70`

Role:

- Database Server
- Application Data Storage


## 3. Network Placement

WEB01과 DB01은 Server Network에 배치한다.

Server Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

WEB01:

`10.10.20.60`

DB01:

`10.10.20.70`

두 서버 모두 Static IP를 사용한다.


## 4. Service Flow

사용자가 사내 웹 서비스를 이용하는 기본 흐름은 다음과 같다.

    User PC
       |
       | HTTP / HTTPS
       ↓
    WEB01
       |
       | Database Connection
       ↓
    DB01
       |
       ↓
    Database

사용자는 DB01에 직접 접근하지 않고 WEB01을 통해 업무 데이터를 조회하거나 입력한다.


## 5. Internal Web Service

이번 프로젝트에서는 실제 제조기업의 내부 업무 환경을 가정하여 간단한 사내 업무 관리 시스템을 구축한다.

가칭:

`SunTech IT Service Portal`

주요 기능은 다음과 같이 구성한다.

- 사내 사용자 로그인
- 공지사항 조회
- IT 장애 / 요청 등록
- 요청 처리 상태 조회
- 간단한 장비 정보 조회


## 6. Web Application Scope

처음부터 복잡한 ERP나 MES를 구현하지 않는다.

이번 프로젝트의 목적은 애플리케이션 개발 자체가 아니라 기업 IT 인프라 환경에서 웹 서비스가 어떻게 운영되는지를 보여주는 것이다.

따라서 다음 수준으로 구성한다.

### 기본 기능

- Login
- Dashboard
- Notice
- IT Request
- Request Status
- Equipment Information


## 7. Authentication

웹 서비스의 로그인은 Active Directory와 연계하는 방안을 검토한다.

기본적인 구조:

    User
      |
      ↓
    WEB01
      |
      ↓
    Active Directory
      |
      ↓
    Authentication

초기 구축 단계에서는 웹 애플리케이션 자체 계정을 이용하여 서비스 기능을 먼저 검증할 수 있다.

이후 AD 인증 연계를 추가하여 기업형 서비스 구조로 확장한다.


## 8. Web Server Software

WEB01은 Windows Server 기반으로 구성한다.

Web Server:

`IIS`

Internet Information Services(IIS)를 사용하여 내부 웹 서비스를 제공한다.

주요 구성 요소:

- IIS Web Server
- HTTP / HTTPS
- Application Hosting
- Access Log


## 9. Web Application Technology

웹 애플리케이션은 현재 보유 기술과 구축 난이도를 고려하여 구성한다.

초기 구현은 다음 기술을 우선 검토한다.

- HTML
- CSS
- JavaScript
- ASP.NET Core 또는 간단한 서버 애플리케이션

실제 구축 단계에서 VM 자원과 구현 난이도를 고려하여 하나의 기술 스택으로 확정한다.


## 10. Web Service URL

내부 DNS를 이용하여 Web Server에 접근할 수 있도록 구성한다.

예상 주소:

`http://portal.suntech.local`

또는 HTTPS 적용 후:

`https://portal.suntech.local`

DNS Record:

    portal.suntech.local
            ↓
       10.10.20.60

사용자는 IP 주소 대신 내부 DNS 이름을 통해 웹 서비스에 접근한다.

## 11. Database Server

DB01은 웹 애플리케이션에서 사용하는 데이터를 중앙에서 저장한다.

Hostname:

`DB01`

IP Address:

`10.10.20.70`

Database Server는 WEB01과 분리하여 운영한다.

이를 통해 Web Server 장애와 Database Server 장애를 독립적으로 관리할 수 있도록 한다.


## 12. Database Software

Database는 오픈소스 관계형 데이터베이스를 사용한다.

우선 다음 환경을 검토한다.

`PostgreSQL`

PostgreSQL은 다음과 같은 이유로 선정한다.

- 오픈소스
- 관계형 데이터베이스
- Windows Server 환경에서 운영 가능
- SQL 기반 데이터 관리
- 실습 및 포트폴리오 구축에 적합


## 13. Database Structure

Database 이름:

`suntech_portal`

주요 Table은 다음과 같이 구성한다.

- users
- notices
- service_requests
- equipment
- request_history


## 14. Users Table

웹 서비스 사용자를 관리한다.

주요 Column:

| Column | Type | Description |
|---|---|---|
| user_id | BIGINT | User Identifier |
| employee_no | VARCHAR | Employee Number |
| name | VARCHAR | User Name |
| department | VARCHAR | Department |
| email | VARCHAR | Email |
| role | VARCHAR | User Role |
| created_at | TIMESTAMP | Created Time |

사용자 계정 정보는 향후 AD 인증 연계 시 AD 계정과 연결할 수 있도록 설계한다.


## 15. Notices Table

사내 공지사항을 저장한다.

주요 Column:

| Column | Type | Description |
|---|---|---|
| notice_id | BIGINT | Notice Identifier |
| title | VARCHAR | Notice Title |
| content | TEXT | Notice Content |
| author | VARCHAR | Author |
| created_at | TIMESTAMP | Created Time |
| updated_at | TIMESTAMP | Updated Time |


## 16. Service Requests Table

IT 장애 및 서비스 요청을 저장한다.

주요 Column:

| Column | Type | Description |
|---|---|---|
| request_id | BIGINT | Request Identifier |
| employee_no | VARCHAR | Requester |
| category | VARCHAR | Request Category |
| title | VARCHAR | Request Title |
| description | TEXT | Request Details |
| status | VARCHAR | Request Status |
| priority | VARCHAR | Priority |
| created_at | TIMESTAMP | Created Time |
| updated_at | TIMESTAMP | Updated Time |


## 17. Equipment Table

사내 IT 장비 정보를 관리한다.

주요 Column:

| Column | Type | Description |
|---|---|---|
| equipment_id | BIGINT | Equipment Identifier |
| asset_no | VARCHAR | Asset Number |
| equipment_type | VARCHAR | Equipment Type |
| hostname | VARCHAR | Hostname |
| ip_address | VARCHAR | IP Address |
| department | VARCHAR | Department |
| status | VARCHAR | Equipment Status |
| assigned_user | VARCHAR | Assigned User |


## 18. Request History Table

IT 요청의 처리 이력을 저장한다.

주요 Column:

| Column | Type | Description |
|---|---|---|
| history_id | BIGINT | History Identifier |
| request_id | BIGINT | Request Identifier |
| status | VARCHAR | Status |
| comment | TEXT | Processing Comment |
| processed_by | VARCHAR | Processor |
| created_at | TIMESTAMP | Processing Time |

이를 통해 하나의 요청이 접수부터 처리 완료까지 어떻게 변경되었는지 확인할 수 있다.


## 19. Database Relationship

주요 데이터 관계는 다음과 같이 구성한다.

    users
       |
       | employee_no
       ↓
    service_requests
       |
       | request_id
       ↓
    request_history

사용자 한 명이 여러 개의 IT 요청을 등록할 수 있고, 하나의 요청에는 여러 개의 처리 이력이 존재할 수 있다.


## 20. Web and Database Connection

WEB01은 DB01의 Database에 접속한다.

구조:

    WEB01
       |
       | TCP 5432
       ↓
    DB01
       |
       ↓
    suntech_portal

DB01의 PostgreSQL은 외부 전체 네트워크에 공개하지 않고 WEB01에서 필요한 통신만 허용하도록 구성한다.

## 21. Firewall Policy

Web Server와 Database Server 사이의 통신을 제한한다.

기본 정책:

    User Network
         |
         ↓
       WEB01
         |
         ↓
       DB01

사용자 PC에서 DB01의 PostgreSQL Port에 직접 접근하는 것은 차단한다.

예상 정책:

| Source | Destination | Port | Action |
|---|---|---:|---|
| User Network | WEB01 | 80/443 | Allow |
| WEB01 | DB01 | 5432 | Allow |
| User Network | DB01 | 5432 | Deny |
| Guest Network | WEB01 | 80/443 | Deny |
| Guest Network | DB01 | 5432 | Deny |


## 22. Web Server Security

WEB01에는 다음 보안 설정을 적용한다.

- Windows Firewall
- IIS Access Control
- HTTPS 적용 검토
- 불필요한 IIS 기능 제거
- IIS Access Log 활성화
- 관리자 권한 최소화
- 정기적인 Windows Update


## 23. Database Security

DB01에는 다음 보안 설정을 적용한다.

- Windows Firewall
- Database Authentication
- Database User Permission
- 최소 권한 원칙
- 외부 직접 접근 제한
- Database Backup
- Query / Connection Log 확인


## 24. Database Account

WEB01에서 DB01에 접속할 때 Database Administrator 계정을 사용하지 않는다.

별도의 Application Database Account를 생성한다.

예:

`svc_webapp`

권한:

- 필요한 Database 접근
- 필요한 Table에 대한 SELECT
- 필요한 Table에 대한 INSERT
- 필요한 Table에 대한 UPDATE

불필요한 DROP 또는 Database Administration 권한은 부여하지 않는다.


## 25. Application Connection

WEB01의 애플리케이션은 DB01의 IP 주소를 직접 코드에 저장하지 않고 설정 파일 또는 환경 변수 등을 통해 Database Connection 정보를 관리한다.

예상 구성:

    Application
        |
        ↓
    Configuration
        |
        ↓
    DB01
    10.10.20.70
        |
        ↓
    PostgreSQL
    5432


## 26. Logging

웹 서비스와 데이터베이스의 장애 및 보안 문제를 추적할 수 있도록 로그를 관리한다.

### WEB01

- IIS Access Log
- Application Log
- Windows Event Log

### DB01

- PostgreSQL Log
- Windows Event Log
- Connection Log


## 27. Monitoring Integration

WEB01과 DB01은 추후 MON01과 연계한다.

MON01:

`10.10.20.40`

모니터링 대상:

- Server Availability
- CPU
- Memory
- Disk
- IIS Service
- PostgreSQL Service
- Network Connectivity

기본 구조:

    MON01
      |
      +------ WEB01
      |
      +------ DB01
      |
      +------ FILE01
      |
      +------ DC01
      |
      +------ DC02


## 28. Backup Integration

DB01의 데이터는 BACKUP01에 백업한다.

DB Backup:

    DB01
      |
      ↓
    BACKUP01

백업 대상:

- Database
- Database Configuration
- Application Data

Web Server도 필요한 설정 파일과 애플리케이션 파일을 백업한다.


## 29. Failure Scenario

### WEB01 Failure

WEB01에 장애가 발생하면 사용자가 웹 서비스에 접근할 수 없게 된다.

기본적인 대응:

    WEB01 Failure
         ↓
    Check IIS
         ↓
    Check Windows Service
         ↓
    Check Network
         ↓
    Restore Service


### DB01 Failure

DB01에 장애가 발생하면 WEB01에서 데이터베이스에 접근할 수 없게 된다.

기본적인 대응:

    DB01 Failure
         ↓
    Check PostgreSQL
         ↓
    Check Network
         ↓
    Check Database
         ↓
    Restore Database


## 30. Validation Checklist

### WEB01

- IIS 설치 확인
- IIS Service 정상
- Web Application 정상
- HTTP 접근 확인
- HTTPS 접근 확인
- IIS Log 기록 확인


### DB01

- PostgreSQL 설치 확인
- PostgreSQL Service 정상
- Database 생성 확인
- Table 생성 확인
- Application Account 생성
- WEB01에서 DB 접속 확인


### Network

- User → WEB01 통신 확인
- WEB01 → DB01 통신 확인
- User → DB01 직접 접근 차단
- Guest → Internal Server 접근 차단


### Application

- Login 정상
- Dashboard 정상
- Notice 조회 정상
- IT Request 등록 정상
- Request Status 조회 정상
- Equipment 조회 정상


### Backup

- Database Backup 정상
- Database Restore 정상
- Application 파일 Backup 정상


## 31. Implementation Order

Web / Database 환경은 다음 순서로 구축한다.

1. DB01 VM 생성
2. Windows Server 설치
3. Static IP 설정
4. Domain Join
5. PostgreSQL 설치
6. Database 생성
7. Database Table 생성
8. Application DB Account 생성
9. WEB01 VM 생성
10. Windows Server 설치
11. Static IP 설정
12. Domain Join
13. IIS 설치
14. Web Application 배포
15. Internal DNS Record 생성
16. WEB01 → DB01 연결
17. Firewall Policy 적용
18. Web Service Test
19. Database Connection Test
20. User Access Test
21. Logging 확인
22. Backup 구성
23. Restore Test
24. Monitoring 연계


## 32. Final Design

SunTech의 Web / Database 환경은 역할을 분리하여 구성한다.

    User PC
       |
       | HTTP / HTTPS
       ↓
    WEB01
    10.10.20.60
       |
       | PostgreSQL 5432
       ↓
    DB01
    10.10.20.70
       |
       ↓
    suntech_portal


    Active Directory
       |
       +------ User Authentication
       |
       +------ DNS
       |
       +------ WEB01
       |
       +------ DB01


    BACKUP01
       |
       +------ WEB01 Backup
       |
       +------ DB01 Backup


    MON01
       |
       +------ WEB01 Monitoring
       |
       +------ DB01 Monitoring

Web Server와 Database Server를 분리하여 보안성과 관리성을 확보한다.

사용자는 WEB01을 통해 내부 업무 서비스를 이용하며 DB01에는 직접 접근할 수 없도록 네트워크 정책을 적용한다.

Active Directory와 DNS를 기존 인프라와 연계하고, BACKUP01을 통해 데이터 백업 및 복구 환경을 구성한다.

또한 MON01을 통해 Web Server와 Database Server의 상태를 모니터링하여 기업형 IT 인프라 운영 환경을 구현한다.
