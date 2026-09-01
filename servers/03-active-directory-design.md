# 03. Active Directory Design

## 1. Purpose

SunTech의 사용자, 컴퓨터 및 IT 자원을 중앙에서 관리하기 위해 Active Directory Domain Services(AD DS) 환경을 구축한다.

Active Directory는 사용자 인증을 비롯하여 부서별 사용자 및 컴퓨터 관리, 보안 정책 적용, 파일 서버 접근 권한 관리 등의 기반으로 사용한다.

본 프로젝트에서는 약 200명의 임직원이 근무하는 제조기업을 가정하여 실제 기업 환경에서 활용할 수 있는 구조를 설계한다.


## 2. Active Directory Architecture

Active Directory Domain Services는 두 대의 Domain Controller로 구성한다.

| Server | Role | IP Address |
|---|---|---|
| DC01 | First Domain Controller / FSMO Roles / DNS | 10.10.20.10 |
| DC02 | Additional Domain Controller / DNS | 10.10.20.11 |

DC01에서 초기 Active Directory Domain을 생성하고 FSMO 역할을 보유하도록 구성한다.

DC02는 DC01의 Additional Domain Controller로 추가하여 Active Directory 데이터를 복제한다.

이를 통해 하나의 Domain Controller에 장애가 발생하더라도 다른 Domain Controller를 이용하여 인증 서비스를 지속할 수 있도록 한다.


## 3. Domain Name

프로젝트의 내부 Active Directory 도메인은 다음과 같이 구성한다.

### Internal Domain

`suntech.local`

### NetBIOS Name

`SUNTECH`

기존 프로젝트와 동일하게 SunTech의 내부 Active Directory 도메인으로 `suntech.local`을 사용한다.

`SUNTECH`는 해당 도메인의 NetBIOS 이름으로 사용한다.

도메인 이름은 실습 환경에서 사용하는 내부 전용 도메인으로 구성한다.

실제 기업 환경에서는 보유한 공식 도메인과 내부 DNS 설계 정책에 따라 별도의 도메인 구성을 사용할 수 있다.


## 4. Domain Controller Roles

### DC01

DC01은 첫 번째 Domain Controller로 구성한다.

주요 역할:

- Active Directory Domain Services
- DNS Server
- Global Catalog
- FSMO Roles
- User Authentication

초기 Active Directory Forest 및 Domain은 DC01에서 생성한다.


### DC02

DC02는 DC01의 Additional Domain Controller로 구성한다.

주요 역할:

- Active Directory Domain Services
- DNS Server
- Global Catalog
- User Authentication
- Active Directory Replication

DC02는 DC01과 동일한 도메인에 추가하여 Active Directory 데이터를 복제한다.


## 5. DNS Design

Active Directory 환경에서는 DNS를 핵심 서비스로 사용한다.

DNS Server:

- DC01
- DC02

주요 역할:

- Domain Name Resolution
- Active Directory Service Discovery
- Client Name Resolution
- Server Name Resolution

주요 내부 도메인:

`suntech.local`

예상되는 주요 DNS 이름:

- dc01.suntech.local
- dc02.suntech.local
- file01.suntech.local
- web01.suntech.local
- db01.suntech.local
- mail01.suntech.local


## 6. Organizational Unit Design

사용자 및 컴퓨터를 체계적으로 관리하기 위해 역할과 조직 구조를 기준으로 OU를 설계한다.

기본 구조는 다음과 같다.

- SUNTECH
  - Users
  - Computers
  - Groups
  - Servers
  - Admins
  - Service Accounts


### Users

사용자 계정을 부서별로 구분한다.

- Users
  - Management
  - IT
  - Production
  - Quality
  - RnD
  - Sales
  - HR
  - Finance


### Computers

사용자 PC를 부서별로 관리한다.

- Computers
  - Management
  - IT
  - Production
  - Quality
  - RnD
  - Sales
  - HR
  - Finance


### Servers

서버 컴퓨터 계정은 별도의 OU에서 관리한다.

- Servers
  - Domain Controllers
  - Infrastructure
  - File
  - Application
  - Management


### Admins

관리자 계정을 일반 사용자 계정과 분리하여 관리한다.

- Admins
  - Domain Admins
  - IT Admins


### Service Accounts

서비스에서 사용하는 계정을 일반 사용자 계정과 분리한다.

- Service Accounts
  - Web
  - Database
  - Monitoring
  - Backup
  - Mail


## 7. Department Structure

약 200명의 임직원을 다음과 같은 부서로 구성한다.

| Department | Example Role |
|---|---|
| Management | Management |
| IT | IT Infrastructure / System |
| Production | Production / Manufacturing |
| Quality | Quality Control |
| RnD | Research & Development |
| Sales | Sales |
| HR | Human Resources |
| Finance | Finance / Accounting |

실제 사용자는 각 부서 OU에 배치한다.


## 8. User Account Naming Convention

사용자 계정은 사번 기반의 일관된 규칙을 적용한다.

### Account Format

`SUN0001 ~ SUN0200`

예시:

- SUN0001
- SUN0002
- SUN0003

약 200명의 임직원을 가정하여 SUN0001부터 SUN0200까지의 계정 체계를 사용한다.

사번을 계정명으로 사용하여 동명이인 문제를 방지하고 사용자 계정을 일관되게 관리한다.


## 9. User Account Attributes

사용자 계정의 로그인 ID는 사번을 사용하지만, 사용자 정보는 Active Directory의 별도 속성으로 관리한다.

예시:

| Attribute | Example |
|---|---|
| Username | SUN0001 |
| Employee ID | SUN0001 |
| Display Name | 김민수 |
| Department | Production |
| Title | 생산관리 |
| E-mail | 추후 Mail Service와 연계 |

사용자 계정명과 실제 사용자 정보를 분리하여 관리한다.


## 10. Computer Naming Convention

사용자 PC는 부서와 장비 유형을 식별할 수 있도록 이름을 구성한다.

예시:

- IT-PC-001
- PROD-PC-001
- PROD-PC-002
- QA-PC-001
- RND-PC-001
- SALES-PC-001
- HR-PC-001
- FIN-PC-001

서버는 별도의 Hostname 규칙을 사용한다.

예시:

- DC01
- DC02
- DHCP01
- FILE01
- BACKUP01
- MON01
- LOG01
- WEB01
- DB01
- MAIL01


## 11. Security Group Design

파일 서버 및 시스템 권한 관리를 위해 Security Group을 사용한다.

그룹은 부서 및 역할을 기준으로 구성한다.

예시:

- GG-IT-Users
- GG-Production-Users
- GG-Quality-Users
- GG-RnD-Users
- GG-Sales-Users
- GG-HR-Users
- GG-Finance-Users


## 12. File Server Permission Model

FILE01의 부서별 파일 접근 권한은 Active Directory Security Group과 연계한다.

기본적인 권한 구조는 AGDLP 모델을 적용한다.

### AGDLP

Account

→ Global Group

→ Domain Local Group

→ Permission


예시:

사용자 SUN0001

→ GG-Production-Users

→ DL-Production-Share-RW

→ FILE01 Production Folder


### Example Groups

Global Group:

- GG-Production-Users
- GG-Quality-Users
- GG-RnD-Users
- GG-Sales-Users
- GG-HR-Users
- GG-Finance-Users

Domain Local Group:

- DL-Production-Share-RW
- DL-Quality-Share-RW
- DL-RnD-Share-RW
- DL-Sales-Share-RW
- DL-HR-Share-RW
- DL-Finance-Share-RW


## 13. Group Policy Design

Group Policy를 이용하여 사용자 및 컴퓨터의 보안 정책을 중앙에서 관리한다.

초기 GPO는 다음 정책을 중심으로 구성한다.

### Security Policy

- Password Policy
- Account Lockout Policy
- Password Complexity


### User Environment

- Desktop Configuration
- Screen Lock
- Control Panel Restrictions


### Device Control

- USB Storage Control


### Audit Policy

- Logon Events
- Account Management
- File Access


GPO는 전체 도메인에 무조건 적용하지 않고 필요한 OU를 대상으로 단계적으로 적용한다.


## 14. GPO Structure

주요 GPO는 다음과 같이 구성한다.

| GPO Name | Purpose | Target |
|---|---|---|
| GPO-Password-Policy | Password Security | Domain |
| GPO-Account-Lockout | Account Lockout | Domain |
| GPO-USB-Control | USB Storage Control | User Computers |
| GPO-Screen-Lock | Automatic Screen Lock | User Computers |
| GPO-Audit-Policy | Security Audit | Servers / Computers |


## 15. Administrator Account Design

관리자 계정과 일반 사용자 계정을 분리한다.

일반 사용자 계정은 업무용으로 사용한다.

관리자 계정은 서버 및 시스템 관리 작업에만 사용한다.

예시:

일반 사용자 계정:

`SUN0001`

관리 계정:

`adm-sun0001`

관리자 계정에는 일반 사용자보다 높은 권한이 부여되므로 별도의 계정 관리 및 보안 정책을 적용한다.


## 16. Service Account Design

서비스에서 사용하는 계정은 일반 사용자 계정과 분리한다.

예상 서비스 계정:

- svc-web
- svc-db
- svc-monitor
- svc-backup
- svc-mail

서비스 계정에는 서비스 운영에 필요한 최소한의 권한만 부여한다.

가능한 경우 서비스별 계정을 분리하여 하나의 계정이 여러 서비스에 과도한 권한을 갖지 않도록 한다.


## 17. Server OU Design

서버는 역할별 OU로 구분하여 관리한다.

예시:

- Servers
  - Domain Controllers
  - Infrastructure
  - File
  - Application
  - Management

서버 역할에 따라 필요한 GPO를 구분하여 적용할 수 있도록 한다.


## 18. Authentication Flow

사용자가 회사 PC에 로그인하면 Active Directory를 통해 인증한다.

기본적인 인증 흐름:

User Account

→ Client PC

→ DNS

→ Domain Controller

→ Active Directory Authentication

→ Group Membership Check

→ GPO Application

→ Resource Access


## 19. File Access Flow

사용자가 파일 서버에 접근하는 경우 다음과 같은 구조를 사용한다.

User Account

→ Global Group

→ Domain Local Group

→ FILE01 Shared Folder

→ NTFS Permission

→ Access Granted / Denied


## 20. Mail Integration

MAIL01은 Active Directory 환경과 연계하는 방향으로 설계한다.

사용자 정보를 기준으로 메일 계정을 관리할 수 있도록 구성하며, 실제 메일 서버 구축 단계에서 가능한 연계 방법을 검토한다.

예상 구조:

AD User

→ Mail Account

→ Mail Service

→ SMTP / IMAP

사용자 계정 예시:

`SUN0001`

메일 주소 및 메일 도메인은 실제 Mail Server 구축 단계에서 별도로 결정한다.

`suntech.local`은 본 프로젝트의 내부 Active Directory 도메인으로 사용하며, 실제 메일 주소 도메인으로 사용할지는 Mail Service 설계 단계에서 별도로 검토한다.


## 21. Active Directory Security Principles

Active Directory 환경에는 다음 보안 원칙을 적용한다.

1. 최소 권한 원칙
2. 관리자 계정과 일반 계정 분리
3. 서비스 계정 분리
4. 부서별 Security Group 사용
5. GPO를 통한 중앙 보안 정책 적용
6. 계정 잠금 정책
7. 강력한 비밀번호 정책
8. 감사 로그 활성화
9. 서버 OU 분리
10. 불필요한 권한 제거


## 22. Active Directory Implementation Scope

이번 프로젝트에서는 다음 항목을 실제로 구축한다.

- Active Directory Domain Services
- DNS
- Domain Controller 2대
- Active Directory Replication
- Organizational Unit
- User Account
- Security Group
- Computer Account
- Group Policy
- File Server Permission Integration
- Basic Audit Policy
- Administrator Account Separation
- Service Account Management


## 23. Implementation Order

Active Directory 구축은 다음 순서로 진행한다.

1. DC01 VM 생성
2. Windows Server 설치
3. 고정 IP 설정
4. DNS 구성
5. AD DS 설치
6. `suntech.local` Domain 생성
7. DC01 Domain Controller 구성
8. DC02 VM 생성
9. DC02 Domain Join
10. Additional Domain Controller 구성
11. Active Directory Replication 확인
12. OU 생성
13. Security Group 생성
14. User Account 생성
15. Computer Account 구성
16. GPO 구성
17. Client Domain Join
18. File Server와 Security Group 연계
19. Authentication Test
20. Permission Test
21. GPO Application Test
22. 장애 및 복구 테스트


## 24. Validation Checklist

Active Directory 구축 후 다음 항목을 확인한다.

### Domain

- Domain Join 정상 여부
- User Authentication 정상 여부
- DNS Name Resolution 정상 여부


### Domain Controller

- DC01 정상 동작
- DC02 정상 동작
- AD Replication 정상 여부
- DNS Replication 정상 여부


### User

- User Account 생성
- Employee ID 설정
- Group Membership 적용
- Login Test


### GPO

- Password Policy 적용
- Account Lockout 적용
- USB Control 적용
- Screen Lock 적용
- Audit Policy 적용


### File Server

- 부서별 접근 권한 적용
- 허용된 사용자 접근 가능
- 허용되지 않은 사용자 접근 차단


## 25. Design Principles

Active Directory는 SunTech IT Infrastructure의 중앙 인증 및 관리 기반으로 사용한다.

AD DS, DNS, DHCP, File Server, GPO, Monitoring, Logging, Web, Database 및 Mail Service를 단계적으로 연계하여 전체 IT 인프라를 통합 관리할 수 있는 구조를 구축한다.

본 프로젝트에서는 단순히 Active Directory를 설치하는 것보다 사용자 관리, 보안 정책, 파일 권한 및 다른 인프라 서비스와의 연계를 중점적으로 구현한다.

기존에 구축했던 Active Directory 경험을 기반으로 하되, 이번 프로젝트에서는 약 200명 규모의 제조기업 환경을 가정하여 부서별 OU, 사번 기반 계정, 관리자 및 서비스 계정 분리, GPO, 파일 서버 권한, 웹/DB/메일 서비스 연계까지 확장하여 설계한다.
