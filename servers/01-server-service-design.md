# 01. Server & Service Design

## 1. Server Design Overview

SunTech의 IT 인프라는 약 200명의 임직원이 사용하는 제조기업 환경을 기준으로 설계한다.

서버는 핵심 인프라, 운영 관리, 업무 서비스의 세 영역으로 구분하며, 각 서버의 역할을 명확하게 분리한다.

전체 서버는 Server VLAN(`10.10.20.0/24`)에 배치하고, 주요 서버에는 고정 IP 주소를 사용한다.

서버 구성은 실제 VMware 가상화 환경에서 구축 및 테스트할 수 있는 수준을 목표로 한다.


## 2. Server Classification

### 2.1 Core Infrastructure

기업의 기본적인 IT 환경을 운영하기 위해 필요한 핵심 서버이다.

- DC01
- DC02
- DHCP01
- FILE01
- BACKUP01


### 2.2 Operations Management

서버 및 네트워크 상태를 확인하고 운영 로그를 관리하기 위한 서버이다.

- MON01
- LOG01


### 2.3 Business Services

사용자가 실제로 이용할 수 있는 업무 서비스를 제공하기 위한 서버이다.

- WEB01
- DB01
- MAIL01


## 3. Server List

| Hostname | Role | Main Function | Priority |
|---|---|---|---|
| DC01 | Domain Controller | AD DS / DNS | Critical |
| DC02 | Domain Controller | AD DS / DNS | Critical |
| DHCP01 | DHCP Server | IP Address Assignment | Critical |
| FILE01 | File Server | File Sharing / FSRM | Critical |
| BACKUP01 | Backup Server | Server Backup | Critical |
| MON01 | Monitoring Server | Infrastructure Monitoring | High |
| LOG01 | Log Server | Central Log Collection | High |
| WEB01 | Web Server | Internal Web Service | Medium |
| DB01 | Database Server | Application Database | Medium |
| MAIL01 | Mail Server | Internal Mail Service | Medium |


## 4. DC01

### Role

- Active Directory Domain Services
- DNS Server

### Purpose

SunTech의 중앙 사용자 및 컴퓨터 계정 관리와 내부 DNS 서비스를 제공한다.

### Main Functions

- Active Directory Domain Services
- User Account Management
- Computer Account Management
- Group Management
- OU Management
- DNS

DC01은 도메인 환경의 핵심 서버로 사용한다.


## 5. DC02

### Role

- Additional Domain Controller
- DNS Server

### Purpose

DC01의 장애에 대비하여 Active Directory 및 DNS 서비스를 이중화한다.

### Main Functions

- Active Directory Replication
- User Authentication
- DNS
- Domain Controller Redundancy

DC02는 DC01과 동일한 도메인에 추가하여 AD 데이터가 복제되도록 구성한다.


## 6. DHCP01

### Role

- DHCP Server

### Purpose

사용자 및 클라이언트 장비에 IP 주소와 네트워크 설정을 자동으로 할당한다.

### Main Functions

- IP Address Assignment
- Subnet Mask Assignment
- Default Gateway Assignment
- DNS Server Assignment
- DHCP Scope Management

현재 DHCP Scope는 다음 네트워크를 대상으로 구성한다.

- HQ User: `10.10.10.0/24`
- Factory: `10.10.40.0/24`
- IT Management: `10.10.30.0/24`
- Guest: `10.10.50.0/24`

Guest 네트워크의 DHCP는 향후 네트워크 장비 구성에 따라 방화벽 또는 별도 DHCP 서비스로 운영할 수 있다.


## 7. FILE01

### Role

- File Server
- File Server Resource Manager

### Purpose

부서별 업무 파일을 중앙에서 저장하고 관리한다.

### Main Functions

- SMB File Sharing
- NTFS Permission
- Share Permission
- Department Folder Management
- FSRM
- File Screening
- Storage Quota
- File Classification

Active Directory Group을 활용하여 부서별 파일 접근 권한을 관리한다.

파일 서버 권한 설계에는 최소 권한 원칙을 적용한다.


## 8. BACKUP01

### Role

- Backup Server

### Purpose

주요 서버의 데이터를 백업하여 장애 및 데이터 손실에 대비한다.

### Main Functions

- Windows Server Backup
- System State Backup
- File Server Backup
- Backup Schedule
- Backup Verification
- Recovery Test

백업은 단순히 설정하는 것에서 끝내지 않고 실제 복구 테스트까지 수행하는 것을 목표로 한다.


## 9. MON01

### Role

- Monitoring Server

### Purpose

서버 및 주요 인프라 장비의 상태를 중앙에서 확인한다.

### Monitoring Targets

- DC01
- DC02
- DHCP01
- FILE01
- BACKUP01
- WEB01
- DB01
- MAIL01
- Network Devices

### Monitoring Items

- CPU Usage
- Memory Usage
- Disk Usage
- Network Traffic
- Server Availability
- Service Status

모니터링 솔루션은 실제 구축 단계에서 VMware 환경과 사용 가능한 리소스를 고려하여 선정한다.


## 10. LOG01

### Role

- Central Log Server

### Purpose

주요 서버 및 시스템에서 발생하는 로그를 중앙에서 수집하고 확인할 수 있는 환경을 구축한다.

### Log Sources

- DC01
- DC02
- FILE01
- WEB01
- DB01
- MAIL01
- Network Devices

### Main Functions

- Central Log Collection
- Log Search
- Event Monitoring
- Basic Event Analysis

초기 구축에서는 복잡한 SIEM 환경을 구성하지 않고 중앙 로그 수집 및 검색 기능을 중심으로 구현한다.


## 11. WEB01

### Role

- Internal Web Server

### Purpose

임직원이 내부 네트워크에서 사용할 수 있는 간단한 웹 서비스를 제공한다.

### Example Services

- 사내 IT 공지
- 시스템 상태 확인
- 간단한 업무 관리 기능

WEB01은 외부 공개 서비스가 아닌 내부 업무 서비스를 대상으로 구축한다.

웹 서비스 구축은 핵심 인프라가 완성된 이후 진행한다.


## 12. DB01

### Role

- Database Server

### Purpose

WEB01에서 사용하는 데이터를 저장하는 데이터베이스 서버를 구성한다.

### Example Data

- 사용자 정보
- 시스템 정보
- 업무 데이터
- 서비스 로그

DB01은 WEB01과 연계하여 기본적인 웹 애플리케이션과 데이터베이스 간 통신을 구현한다.

사용할 데이터베이스 제품은 실제 구축 단계에서 프로젝트 난이도와 기존 경험을 고려하여 선정한다.


## 13. MAIL01

### Role

- Mail Server

### Purpose

기업 내부 사용자가 사용할 수 있는 메일 서비스를 구축한다.

메일 서버는 이번 프로젝트에서 실제 서비스 구축 경험을 확보하기 위한 업무 서비스로 구성한다.

### Main Functions

- Mail Account Management
- SMTP
- IMAP
- Internal Mail Delivery
- Mail Sending and Receiving Test
- Mail Server Log Management

### Additional Security

메일 서비스가 정상적으로 구축된 이후 다음 기능을 단계적으로 검토한다.

- TLS
- SPF
- DKIM
- DMARC
- SMTP Relay Protection

고급 메일 보안 기능은 기본적인 메일 송수신 환경이 정상적으로 구축된 이후 추가한다.


## 14. Server Dependency

각 서버의 구축 순서는 서버 간 의존성을 고려하여 결정한다.

| Server | Dependency |
|---|---|
| DC01 | None |
| DC02 | DC01 |
| DHCP01 | DC01 / DNS |
| FILE01 | AD / DNS |
| BACKUP01 | 대상 서버 |
| MON01 | Network / Server |
| LOG01 | Log Source Servers |
| WEB01 | AD / DNS |
| DB01 | WEB01 |
| MAIL01 | AD / DNS |


## 15. Recommended Build Order

실제 VMware 환경에서는 다음 순서로 구축한다.

1. DC01
2. DC02
3. DHCP01
4. FILE01
5. BACKUP01
6. MON01
7. LOG01
8. WEB01
9. DB01
10. MAIL01

핵심 인프라를 먼저 완성한 후 운영 관리 및 업무 서비스를 추가한다.


## 16. Virtualization Strategy

모든 서버는 VMware 기반 가상머신으로 구축한다.

가상화 환경을 사용하여 실제 물리 서버를 여러 대 준비하지 않고도 기업 IT 인프라의 주요 기능을 독립된 서버 단위로 구성하고 테스트할 수 있도록 한다.

각 가상머신은 역할별로 분리하여 구성하며, 서버 간 네트워크 통신은 앞서 정의한 VLAN 및 IP Address Plan을 기준으로 한다.


## 17. Resource Allocation Principles

가상머신의 CPU, Memory 및 Storage는 서버 역할과 실제 PC의 가용 자원을 고려하여 할당한다.

초기 구축에서는 최소한의 리소스로 서버를 구성하고, 서비스 테스트 및 모니터링 결과에 따라 필요한 서버의 리소스를 조정한다.

모든 서버에 동일한 사양을 적용하지 않고 역할에 따라 차등적으로 할당한다.


## 18. Server Design Principles

SunTech 서버 환경은 다음 원칙을 기준으로 설계한다.

1. 역할별 서버 분리
2. 핵심 서비스 이중화
3. 최소 권한 원칙
4. 중앙 관리
5. 백업 및 복구
6. 모니터링
7. 중앙 로그 관리
8. 단계적인 서비스 확장
9. VMware 기반 가상화
10. 실제 운영 환경을 고려한 구성


## 19. Future Expansion

향후 기업 규모 및 서비스 요구사항 증가에 따라 다음과 같은 서버 또는 서비스를 추가할 수 있다.

- Additional Domain Controller
- Additional File Server
- Application Server
- Database Server
- Security Management Server
- Additional Monitoring Server
- Additional Log Management
- Production System Server

추가 서버가 필요한 경우 기존 네트워크 및 IP Address Plan과의 연계성을 검토한 후 추가한다.
