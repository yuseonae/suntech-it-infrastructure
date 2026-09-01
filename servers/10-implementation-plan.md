# 10. Implementation Plan

## 1. Purpose

본 문서는 SunTech IT Infrastructure 설계를 실제 VMware 가상 환경에 구축하기 위한 전체 Implementation Plan이다.

앞서 작성한 서버, 네트워크, 보안, 메일, 모니터링, 로그 및 백업 설계를 실제 환경으로 구현하고 각 단계별 구축 결과를 검증한다.

프로젝트 진행 과정에서 본 문서를 구축 체크리스트로 활용한다.

전체 구축 과정은 다음 순서로 진행한다.

Design
→ Build
→ Configure
→ Test
→ Document
→ Validate


## 2. Implementation Environment

### Virtualization

- VMware Workstation Pro

### Windows Server

- Windows Server 2019

### Linux Server

- Linux Distribution
- MAIL01
- 필요에 따라 MON01 / LOG01 구성

### Network

- Server Network: 10.10.20.0/24
- Client Network: 10.10.10.0/24

### Domain

- Domain: suntech.local

### Organization

- Company: SunTech
- Employees: Approximately 200


## 3. Implementation Status

각 단계의 상태는 다음 기준으로 관리한다.

| Status | Meaning |
|---|---|
| PLANNED | 구축 예정 |
| IN PROGRESS | 현재 구축 중 |
| TESTING | 테스트 중 |
| COMPLETED | 구축 및 테스트 완료 |
| ISSUE | 문제 발생 및 해결 필요 |

실제 구축이 완료되고 검증까지 끝난 항목만 COMPLETED로 변경한다.


## 4. Phase 1 - VMware Environment

### 4.1 VMware Preparation

- VMware Workstation Pro 설치 확인
- VM 저장 위치 확인
- Virtual Network 구성
- NAT / Host-only Network 확인
- VM Resource 기준 확인

Status:

`PLANNED`


### 4.2 Virtual Machine Creation

생성할 VM:

| VM | Role | OS |
|---|---|---|
| DC01 | Primary Domain Controller | Windows Server 2019 |
| DC02 | Additional Domain Controller | Windows Server 2019 |
| DHCP01 | DHCP Server | Windows Server 2019 |
| FILE01 | File Server | Windows Server 2019 |
| BACKUP01 | Backup Server | Windows Server 2019 |
| MON01 | Monitoring Server | Linux |
| LOG01 | Log Server | Linux |
| WEB01 | Web Server | Windows Server 2019 |
| DB01 | Database Server | Windows Server 2019 |
| MAIL01 | Mail Server | Linux |

Status:

`PLANNED`


## 5. Phase 2 - Network Foundation

### 5.1 Network Configuration

각 서버에 Static IP를 설정한다.

Server Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

DNS:

- Primary: 10.10.20.10
- Secondary: 10.10.20.11

단, DC01 구축 초기에는 DNS 구성 순서에 따라 DNS 설정을 단계적으로 적용한다.

Status:

`PLANNED`


### 5.2 Basic Connectivity Test

각 서버 간 기본 통신을 확인한다.

Test:

- Ping
- IP Configuration
- Gateway
- DNS Resolution
- Hostname Resolution

Status:

`PLANNED`


## 6. Phase 3 - Active Directory

### 6.1 DC01

구축 항목:

- Windows Server Installation
- Hostname Configuration
- Static IP
- Active Directory Domain Services
- DNS Server
- Domain Controller Promotion
- Domain: suntech.local

Status:

`PLANNED`


### 6.2 Active Directory Structure

OU 구조를 생성한다.

예:

SunTech

- Users
- Groups
- Computers
- Servers
- Workstations
- Departments
  - Production
  - Quality
  - IT
  - Sales


Status:

`PLANNED`


### 6.3 User and Group

약 200명의 사용자를 가정하여 사용자 계정 구조를 구성한다.

사용자 ID:

`SUN0001`

형식:

`SUN + 4자리 사번`

예:

- SUN0001
- SUN0002
- SUN0003
- SUN0200

부서별 Security Group을 생성한다.

예:

- GG-Production
- GG-Quality
- GG-IT
- GG-Sales


Status:

`PLANNED`


### 6.4 GPO

주요 Group Policy를 구성한다.

- Password Policy
- Account Lockout Policy
- USB Storage Control
- Screen Lock
- Security Policy

Status:

`PLANNED`


### 6.5 DC02

DC02를 Additional Domain Controller로 구성한다.

구축 항목:

- Windows Server Installation
- Static IP
- DNS
- Domain Join
- AD DS Installation
- Additional Domain Controller Promotion
- Replication Test

Status:

`PLANNED`


## 7. Phase 4 - DHCP / DNS

### 7.1 DHCP01

DHCP Server를 구축한다.

구축 항목:

- DHCP Server Role
- Scope Creation
- Address Pool
- Exclusion
- Reservation
- DHCP Options


Status:

`PLANNED`


### 7.2 DHCP Scope

Client Network:

`10.10.10.0/24`

예상 범위:

`10.10.10.100 - 10.10.10.200`

Gateway:

`10.10.10.1`

DNS:

`10.10.20.10`

Status:

`PLANNED`


### 7.3 DHCP Test

테스트 PC에서:

- Automatic IP
- IP Assignment
- Gateway Assignment
- DNS Assignment
- Domain Resolution

확인

Status:

`PLANNED`


### 7.4 DNS Test

확인 항목:

- Forward Lookup
- Reverse Lookup
- Server Name Resolution
- Domain Resolution
- WEB01 Resolution
- DB01 Resolution
- MAIL01 Resolution

Status:

`PLANNED`


## 8. Phase 5 - File Server

### 8.1 FILE01

구축 항목:

- Windows Server Installation
- Static IP
- File Server Role
- Shared Folder
- NTFS Permission
- Share Permission
- Department Structure


Status:

`PLANNED`


### 8.2 File Share

예상 구조:

FILE01

- Production
- Quality
- IT
- Sales

각 부서별 Security Group을 이용하여 접근 권한을 관리한다.

권한 모델:

`AGDLP`

Account
→ Global Group
→ Domain Local Group
→ Permission


Status:

`PLANNED`


### 8.3 FSRM

구축 항목:

- File Server Resource Manager
- File Screening
- Storage Report
- Quota


Status:

`PLANNED`


### 8.4 File Server Test

테스트:

- Production 사용자 접근
- Quality 사용자 접근
- IT 사용자 접근
- Sales 사용자 접근
- 타 부서 접근 차단
- 파일 생성
- 파일 수정
- 파일 삭제
- 권한 상속 확인

Status:

`PLANNED`


## 9. Phase 6 - Backup Server

### 9.1 BACKUP01

구축 항목:

- Windows Server Installation
- Static IP
- Backup Storage
- Windows Server Backup
- Backup Directory
- Access Control


Status:

`PLANNED`


### 9.2 Backup Configuration

Backup Target:

- DC01
- DC02
- DHCP01
- FILE01
- WEB01
- DB01
- MAIL01
- MON01
- LOG01


Status:

`PLANNED`


### 9.3 Backup Test

테스트:

- Backup Job 실행
- Backup Result 확인
- Backup File 확인
- Restore Test


Status:

`PLANNED`


## 10. Phase 7 - Web Server

### 10.1 WEB01

구축 항목:

- Windows Server Installation
- IIS
- Web Application
- HTTPS
- DNS Integration
- Database Connection


Status:

`PLANNED`


### 10.2 Web Application

내부 업무용 Web Application을 구성한다.

예상 기능:

- Login
- Notice
- Service Request
- Equipment
- Request History


Status:

`PLANNED`


### 10.3 Web Test

확인:

- DNS Resolution
- HTTP Access
- HTTPS Access
- Login
- Database Connection
- CRUD Function


Status:

`PLANNED`


## 11. Phase 8 - Database Server

### 11.1 DB01

Database Server를 구축한다.

구축 항목:

- Windows Server Installation
- PostgreSQL
- Database
- User Account
- Permission
- Backup


Status:

`PLANNED`


### 11.2 Database

Database:

`suntech_portal`

주요 Table:

- users
- notices
- service_requests
- equipment
- request_history


Status:

`PLANNED`


### 11.3 Database Test

확인:

- Database Connection
- User Authentication
- SELECT
- INSERT
- UPDATE
- DELETE
- Permission
- Backup
- Restore


Status:

`PLANNED`


## 12. Phase 9 - Mail Server

### 12.1 MAIL01

Linux 기반 Mail Server를 구축한다.

구축 항목:

- Linux Installation
- Static IP
- Postfix
- Dovecot
- TLS
- User Mailbox
- DNS Configuration


Status:

`PLANNED`


### 12.2 Mail Domain

Mail Domain:

`suntech.local`

예상 계정:

`sun0001@suntech.local`

실제 외부 인터넷 메일 서비스가 아닌 내부 IT Infrastructure 실습 환경에서 동작하는 사내 메일 시스템을 목표로 한다.


Status:

`PLANNED`


### 12.3 Mail Test

확인:

- SMTP Connection
- Internal Mail Send
- Internal Mail Receive
- IMAP Connection
- Authentication
- TLS
- Authentication Failure


Status:

`PLANNED`


## 13. Phase 10 - Monitoring Server

### 13.1 MON01

Monitoring Server를 구축한다.

Software:

`Zabbix`

구축 항목:

- Linux Installation
- Zabbix Server
- Zabbix Agent
- Dashboard
- Alert


Status:

`PLANNED`


### 13.2 Monitoring Targets

Monitoring 대상:

- DC01
- DC02
- DHCP01
- FILE01
- BACKUP01
- WEB01
- DB01
- MAIL01
- LOG01


Monitoring:

- Availability
- CPU
- Memory
- Disk
- Network
- Port
- Service


Status:

`PLANNED`


### 13.3 Monitoring Test

장애 상황을 가정한다.

Test:

- Server Down
- Service Down
- High CPU
- Low Disk
- Port Down

Alert 발생 여부를 확인한다.

Status:

`PLANNED`


## 14. Phase 11 - Log Server

### 14.1 LOG01

Central Log Server를 구축한다.

Software:

`Wazuh`

구축 항목:

- Linux Installation
- Wazuh
- Agent
- Dashboard
- Log Collection


Status:

`PLANNED`


### 14.2 Log Collection

수집 대상:

- DC01 Security Log
- DC02 Security Log
- DHCP01 Event Log
- FILE01 Security Log
- WEB01 IIS Log
- DB01 PostgreSQL Log
- MAIL01 Mail Log


Status:

`PLANNED`


### 14.3 Log Test

테스트 이벤트:

- Failed Login
- Account Lockout
- File Access
- File Modification
- IIS Error
- Database Error
- Mail Authentication Failure


Status:

`PLANNED`


## 15. Phase 12 - Security Hardening

각 서버의 기본 보안을 적용한다.

### Account Security

- Strong Password
- Account Lockout
- Administrator Account Management
- Least Privilege


### Network Security

- Windows Firewall
- Linux Firewall
- Unnecessary Port Blocking
- Service Access Restriction


### Server Security

- Disable Unnecessary Services
- Security Updates
- Access Control
- Audit Policy


Status:

`PLANNED`


## 16. Phase 13 - Backup and Recovery

각 서버의 Backup 및 Recovery를 검증한다.

### File Recovery

- File Delete
- Restore
- Permission Check


### Database Recovery

- Data Delete
- Restore
- Application Test


### Web Recovery

- Service Failure
- Configuration Restore
- Application Test


### Mail Recovery

- Mailbox Delete
- Restore
- SMTP / IMAP Test


Status:

`PLANNED`


## 17. Phase 14 - Integrated Test

개별 서버 테스트가 완료되면 전체 Infrastructure를 통합 테스트한다.

### Test 1. User Authentication

Client

→ DC01

→ Authentication

→ GPO

→ User Desktop


Status:

`PLANNED`


### Test 2. IP Assignment

Client

→ DHCP01

→ IP Address

→ DNS

→ Domain


Status:

`PLANNED`


### Test 3. File Access

User

→ FILE01

→ Department Share

→ Permission Check


Status:

`PLANNED`


### Test 4. Web Service

User

→ DNS

→ WEB01

→ DB01

→ Web Application


Status:

`PLANNED`


### Test 5. Mail

User

→ MAIL01

→ SMTP / IMAP

→ Internal Mail


Status:

`PLANNED`


### Test 6. Monitoring

Server Failure

→ MON01

→ Alert


Status:

`PLANNED`


### Test 7. Log Analysis

Security Event

→ LOG01

→ Wazuh

→ Event Analysis


Status:

`PLANNED`


### Test 8. Recovery

Failure

→ BACKUP01

→ Restore

→ Validation


Status:

`PLANNED`


## 18. End-to-End Scenario

최종적으로 실제 기업 환경을 가정한 하나의 시나리오를 수행한다.

### Scenario

신규 직원이 입사한다.

1. AD User Account 생성
2. Department Group 추가
3. Client PC Domain Join
4. DHCP IP Assignment
5. GPO 적용
6. File Server Permission 적용
7. Web Application Login
8. Database Access
9. Mail Account 사용
10. Monitoring 대상 등록
11. Log Collection 확인


그 후 장애 상황을 발생시킨다.

### Failure Scenario

1. WEB01 Service Stop
2. MON01 Alert 발생
3. LOG01에서 Event 확인
4. 원인 분석
5. Service Recovery
6. WEB01 정상 여부 확인


마지막으로 데이터 장애를 발생시킨다.

### Recovery Scenario

1. FILE01 테스트 파일 삭제
2. BACKUP01 Backup 확인
3. File Restore
4. Permission 확인
5. User Access Test
6. Recovery 완료


## 19. Documentation

각 구축 단계가 완료되면 관련 문서를 함께 작성한다.

문서 유형:

- Configuration
- Screenshot
- Test Result
- Troubleshooting
- Recovery Result


문서는 GitHub Repository에 저장한다.


## 20. Evidence Collection

포트폴리오에 활용할 수 있도록 주요 구축 결과를 캡처한다.

### Network

- IP Configuration
- DNS
- DHCP


### Active Directory

- AD Structure
- OU
- Users
- Groups
- GPO


### File Server

- Share
- NTFS Permission
- FSRM


### Web / DB

- IIS
- Web Application
- Database
- Application Connection


### Mail

- Postfix
- Dovecot
- Mail Test


### Monitoring

- Zabbix Dashboard
- Alert


### Logging

- Wazuh Dashboard
- Security Event


### Backup

- Backup Job
- Restore Test


## 21. Troubleshooting Record

구축 과정에서 발생한 문제는 별도로 기록한다.

기록 형식:

Problem

→ Cause

→ Investigation

→ Solution

→ Validation


예:

FILE01 Permission Issue

→ User cannot access department folder

→ Check Share / NTFS / Group Membership

→ Correct Permission

→ User Access Test


실제 구축 중 발생한 문제를 포트폴리오의 Troubleshooting 사례로 활용한다.


## 22. Final Validation

전체 구축이 완료되면 다음 항목을 확인한다.

### Infrastructure

- All VM Running
- Network Connectivity
- DNS
- DHCP


### Authentication

- AD
- Domain Join
- User Login
- GPO


### File

- Share
- NTFS
- AGDLP
- FSRM


### Application

- WEB01
- DB01
- Application
- Database Connection


### Mail

- SMTP
- IMAP
- TLS
- Authentication


### Monitoring

- Zabbix
- Agent
- Dashboard
- Alert


### Logging

- Wazuh
- Windows Event Log
- Linux Log
- Application Log


### Backup

- Backup Job
- Backup Storage
- Restore Test


## 23. Completion Criteria

프로젝트는 다음 조건을 모두 만족했을 때 완료로 판단한다.

1. 모든 주요 VM 구축 완료
2. 네트워크 통신 정상
3. AD 정상
4. DNS 정상
5. DHCP 정상
6. File Server 정상
7. Web Server 정상
8. Database 정상
9. Mail Server 정상
10. Monitoring 정상
11. Log Collection 정상
12. Backup 정상
13. Restore Test 완료
14. End-to-End Test 완료
15. 주요 장애 시나리오 해결
16. GitHub Documentation 완료


## 24. Final Infrastructure Flow

최종적으로 SunTech IT Infrastructure는 다음과 같은 흐름으로 동작한다.

User
→ Client Network
→ DHCP
→ DNS
→ Active Directory
→ File Server
→ Web Server
→ Database
→ Mail Server


Infrastructure Operations:

Servers
→ MON01
→ Monitoring


Servers
→ LOG01
→ Log Analysis


Servers
→ BACKUP01
→ Backup / Recovery


## 25. Project Completion

최종 프로젝트 목표:

"약 200명의 사용자가 근무하는 제조기업 SunTech를 가정하여 Windows Server 기반 Active Directory를 중심으로 네트워크, 파일 서버, 업무용 Web / Database, 사내 Mail, Monitoring, Central Logging 및 Backup / Recovery 환경을 직접 설계하고 구축한다."

단순한 서버 설치가 아니라 다음의 전체 IT Infrastructure 운영 과정을 구현한다.

Design
→ Build
→ Configure
→ Secure
→ Monitor
→ Analyze
→ Backup
→ Recover
→ Validate


이를 통해 실제 제조기업의 IT Infrastructure / System Operation 직무에서 요구되는 서버 운영, 네트워크, 계정 및 권한 관리, 보안, 장애 대응 및 시스템 운영 역량을 하나의 프로젝트로 보여주는 것을 목표로 한다.
