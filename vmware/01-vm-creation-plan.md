# 01. VM Creation Plan

## 1. Purpose

본 문서는 SunTech IT Infrastructure 프로젝트에서 사용하는 VMware Workstation Pro 가상 머신의 생성 기준을 정의한다.

각 Server VM의 이름, 운영체제, CPU, Memory, Storage, Network 및 IP Address를 사전에 정의하여 실제 구축 과정에서 일관된 환경을 유지하는 것을 목적으로 한다.

또한 Host PC의 제한된 자원을 고려하여 모든 VM을 동시에 실행하지 않고 구축 단계에 따라 필요한 VM을 선택적으로 실행한다.


## 2. Host PC Specification

현재 프로젝트의 Host PC 사양은 다음과 같다.

| Component | Specification |
|---|---|
| CPU | Intel Core i7-9700F |
| RAM | 32GB |
| Storage | C: 약 465GB / D: 약 1.81TB |
| Virtualization | VMware Workstation Pro |

VM 저장 공간은 여유 공간과 관리 편의성을 고려하여 가능하면 D: 드라이브를 사용한다.


## 3. Resource Allocation Principle

Host PC의 RAM은 32GB이므로 모든 VM에 높은 사양을 할당하지 않는다.

기본 원칙:

- AD / DNS 등 핵심 서버는 안정성을 우선한다.
- 일반 Infrastructure Server는 최소 운영 가능한 Memory를 사용한다.
- Database / Log Server는 필요에 따라 Memory를 추가한다.
- Client VM은 최소 사양으로 구성한다.
- 사용하지 않는 VM은 종료한다.
- 구축 단계별로 필요한 VM만 동시에 실행한다.

VM에 할당하는 RAM의 총합이 Host PC의 실제 사용 가능 Memory를 과도하게 초과하지 않도록 관리한다.


## 4. Virtual Machine Naming Convention

모든 Server VM은 다음 규칙을 사용한다.

`ROLE + NUMBER`

예:

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

이름만 확인해도 서버의 역할을 쉽게 파악할 수 있도록 구성한다.


## 5. Operating System

### Windows Server

다음 서버는 Windows Server 2019를 사용한다.

- DC01
- DC02
- DHCP01
- FILE01
- BACKUP01
- WEB01
- DB01

### Linux

다음 서버는 Linux 기반으로 구성한다.

- MON01
- LOG01
- MAIL01

Linux Distribution은 실제 구축 단계에서 하나의 배포판으로 통일한다.

## 6. Network Configuration

Server Network:

`10.10.20.0/24`

Client Network:

`10.10.10.0/24`

Server는 기본적으로 Static IP를 사용한다.

Client는 DHCP를 사용한다.


## 7. Domain

Active Directory Domain:

`suntech.local`

Domain Controller:

- DC01
- DC02

DNS:

- DC01: 10.10.20.10
- DC02: 10.10.20.11


## 8. Virtual Machine Specification

| VM | Role | OS | vCPU | RAM | Disk | IP |
|---|---|---|---:|---:|---:|---|
| DC01 | AD / DNS | Windows Server 2019 | 2 | 2GB | 60GB | 10.10.20.10 |
| DC02 | AD / DNS | Windows Server 2019 | 2 | 2GB | 60GB | 10.10.20.11 |
| DHCP01 | DHCP | Windows Server 2019 | 2 | 2GB | 50GB | 10.10.20.12 |
| FILE01 | File Server | Windows Server 2019 | 2 | 3GB | 80GB | 10.10.20.20 |
| BACKUP01 | Backup | Windows Server 2019 | 2 | 3GB | 120GB | 10.10.20.30 |
| MON01 | Monitoring | Linux | 2 | 2GB | 40GB | 10.10.20.40 |
| LOG01 | Logging | Linux | 2 | 4GB | 60GB | 10.10.20.50 |
| WEB01 | Web Server | Windows Server 2019 | 2 | 2GB | 60GB | 10.10.20.60 |
| DB01 | Database | Windows Server 2019 | 2 | 3GB | 60GB | 10.10.20.70 |
| MAIL01 | Mail Server | Linux | 2 | 2GB | 50GB | 10.10.20.80 |


## 9. DC01

### Basic Configuration

Hostname:

`DC01`

Role:

- Active Directory Domain Services
- DNS Server
- Primary Domain Controller

OS:

`Windows Server 2019`

vCPU:

`2`

RAM:

`2GB`

Disk:

`60GB`

IP:

`10.10.20.10`

Gateway:

`10.10.20.1`

DNS:

`10.10.20.10`

DC01은 전체 Infrastructure의 핵심 서버이므로 가장 먼저 구축한다.


## 10. DC02

### Basic Configuration

Hostname:

`DC02`

Role:

- Additional Domain Controller
- DNS Server

OS:

`Windows Server 2019`

vCPU:

`2`

RAM:

`2GB`

Disk:

`60GB`

IP:

`10.10.20.11`

Gateway:

`10.10.20.1`

DNS:

`10.10.20.10`

DC02는 DC01 구축 이후 Domain에 추가한다.

구축 후 Active Directory Replication과 DNS 동작을 확인한다.

## 11. DHCP01

### Basic Configuration

Hostname:

`DHCP01`

Role:

- DHCP Server

OS:

`Windows Server 2019`

vCPU:

`2`

RAM:

`2GB`

Disk:

`50GB`

IP:

`10.10.20.12`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

Client Network에 IP Address를 자동으로 할당한다.


## 12. FILE01

### Basic Configuration

Hostname:

`FILE01`

Role:

- File Server
- FSRM

OS:

`Windows Server 2019`

vCPU:

`2`

RAM:

`3GB`

Disk:

`80GB`

IP:

`10.10.20.20`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

부서별 Shared Folder와 NTFS Permission을 구성한다.


## 13. BACKUP01

### Basic Configuration

Hostname:

`BACKUP01`

Role:

- Backup Server
- Windows Server Backup

OS:

`Windows Server 2019`

vCPU:

`2`

RAM:

`3GB`

Disk:

`120GB`

IP:

`10.10.20.30`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

실습 환경의 Backup 및 Restore Test를 담당한다.

실제 기업 환경의 백업 저장소와 달리 Host PC의 제한된 Disk를 고려하여 프로젝트 규모에 맞게 구성한다.


## 14. MON01

### Basic Configuration

Hostname:

`MON01`

Role:

- Monitoring Server
- Zabbix Server
- Dashboard
- Alert

OS:

`Linux`

vCPU:

`2`

RAM:

`2GB`

Disk:

`40GB`

IP:

`10.10.20.40`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11


## 15. LOG01

### Basic Configuration

Hostname:

`LOG01`

Role:

- Central Log Server
- Wazuh
- Security Event Analysis

OS:

`Linux`

vCPU:

`2`

RAM:

`4GB`

Disk:

`60GB`

IP:

`10.10.20.50`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

Log Server는 다른 서버에서 전송되는 로그를 저장하고 분석한다.

로그 데이터 증가를 고려하여 다른 Linux Server보다 Memory와 Storage를 상대적으로 높게 할당한다.

## 16. WEB01

### Basic Configuration

Hostname:

`WEB01`

Role:

- IIS
- Internal Web Application
- HTTPS

OS:

`Windows Server 2019`

vCPU:

`2`

RAM:

`2GB`

Disk:

`60GB`

IP:

`10.10.20.60`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

내부 업무용 Web Application을 제공한다.


## 17. DB01

### Basic Configuration

Hostname:

`DB01`

Role:

- PostgreSQL
- Application Database

OS:

`Windows Server 2019`

vCPU:

`2`

RAM:

`3GB`

Disk:

`60GB`

IP:

`10.10.20.70`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

WEB01에서 사용하는 Database를 제공한다.


## 18. MAIL01

### Basic Configuration

Hostname:

`MAIL01`

Role:

- Mail Server
- Postfix
- Dovecot
- SMTP
- IMAP

OS:

`Linux`

vCPU:

`2`

RAM:

`2GB`

Disk:

`50GB`

IP:

`10.10.20.80`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

내부 Infrastructure 실습 환경에서 사내 Mail 기능을 구현한다.


## 19. Client VM

Client PC도 별도의 VM으로 구성한다.

Hostname:

`CLIENT01`

OS:

`Windows 10 / Windows 11`

vCPU:

`2`

RAM:

`4GB`

Disk:

`60GB`

Network:

`10.10.10.0/24`

IP:

`DHCP`

DNS:

`10.10.20.10`

Client는 Active Directory Domain Join, GPO, File Server, Web Application 및 Mail 기능을 테스트하는 용도로 사용한다.


## 20. Client Test Accounts

Client VM에서는 실제 기업 사용 환경을 가정하여 여러 사용자 계정을 테스트한다.

예:

- SUN0001
- SUN0002
- SUN0003

부서:

- Production
- Quality
- IT
- Sales

실제 테스트에서는 부서별 권한 차이를 확인한다.

## 21. Disk Allocation Principle

각 VM의 OS Disk는 기본적으로 하나의 Virtual Disk로 구성한다.

단, File Server와 Backup Server는 데이터 저장 공간을 별도의 Virtual Disk로 분리하는 방식을 우선 검토한다.

예:

FILE01:

OS Disk:

`C: 60GB`

Data Disk:

`D: 별도 구성`

BACKUP01:

OS Disk:

`C: 60GB`

Backup Disk:

`D: 별도 구성`

이렇게 구성하면 OS 영역과 데이터 영역을 분리하여 관리할 수 있다.


## 22. Host Resource Management

Host PC의 RAM은 32GB이므로 모든 VM을 동시에 실행하지 않는다.

권장 운영 방식:

### Always On

항상 실행할 핵심 VM:

- DC01
- DC02

### Infrastructure

필요한 경우 실행:

- DHCP01
- FILE01

### Application

Web / Database 테스트 시 실행:

- WEB01
- DB01

### Operation

모니터링 / 로그 테스트 시 실행:

- MON01
- LOG01

### Additional Services

메일 테스트 시 실행:

- MAIL01

### Recovery

백업 테스트 시 실행:

- BACKUP01


## 23. Recommended Concurrent VM Groups

### Group A - AD / Network

실행:

- DC01
- DC02
- DHCP01
- CLIENT01

목적:

- AD
- DNS
- DHCP
- Domain Join
- GPO

예상 Memory:

약 `10GB`


### Group B - File / Backup

실행:

- DC01
- DC02
- FILE01
- BACKUP01
- CLIENT01

목적:

- File Server
- NTFS Permission
- AGDLP
- Backup
- Restore

예상 Memory:

약 `14GB`


### Group C - Web / Database

실행:

- DC01
- DC02
- WEB01
- DB01
- CLIENT01

목적:

- IIS
- Web Application
- PostgreSQL
- Application ↔ Database

예상 Memory:

약 `13GB`


### Group D - Monitoring / Logging

실행:

- DC01
- MON01
- LOG01
- CLIENT01

목적:

- Zabbix
- Wazuh
- Monitoring
- Log Collection

예상 Memory:

약 `12GB`


### Group E - Mail

실행:

- DC01
- MAIL01
- CLIENT01

목적:

- SMTP
- IMAP
- Internal Mail

예상 Memory:

약 `10GB`


## 24. VM Startup Order

VM은 다음 순서로 구축 및 실행한다.

### Step 1

`DC01`

### Step 2

`DC02`

### Step 3

`DHCP01`

### Step 4

`FILE01`

### Step 5

`BACKUP01`

### Step 6

`WEB01`

### Step 7

`DB01`

### Step 8

`MAIL01`

### Step 9

`MON01`

### Step 10

`LOG01`

### Step 11

`CLIENT01`


## 25. VM Storage Location

VM 파일은 가능하면 D: Drive에 저장한다.

예상 구조:

`D:\VMware\SunTech\`

하위 폴더:

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
- CLIENT01

VM 이름과 Folder 이름을 동일하게 유지하여 관리 편의성을 높인다.

## 26. VM Snapshot Policy

Snapshot은 주요 변경 직전에 제한적으로 사용한다.

Snapshot을 생성할 시점:

- OS 설치 완료
- AD 구축 완료
- 주요 Server Role 설치 전
- 주요 설정 변경 전

Snapshot을 무분별하게 장기간 유지하지 않는다.

특히 Backup을 Snapshot의 대체 수단으로 사용하지 않는다.


## 27. VM Creation Checklist

### Common

- [ ] VM Name
- [ ] OS
- [ ] CPU
- [ ] RAM
- [ ] Disk
- [ ] Network Adapter
- [ ] VM Folder

### Network

- [ ] Virtual Network 연결
- [ ] IP Configuration
- [ ] Gateway
- [ ] DNS
- [ ] Hostname

### Windows Server

- [ ] Windows Installation
- [ ] VMware Tools
- [ ] Windows Update
- [ ] Hostname
- [ ] Static IP

### Linux

- [ ] Linux Installation
- [ ] VMware Tools / Guest Integration
- [ ] Hostname
- [ ] Static IP
- [ ] DNS
- [ ] Firewall


## 28. Evidence Collection

각 VM 생성 및 초기 설정 과정에서 다음 화면을 캡처한다.

- VM Hardware Settings
- CPU / RAM
- Disk
- Network Adapter
- Operating System
- Hostname
- IP Configuration

캡처한 자료는 추후 GitHub Documentation 및 Portfolio에 활용한다.


## 29. Resource Adjustment Policy

실제 구축 과정에서 Application의 요구사항에 따라 Resource를 조정할 수 있다.

예:

LOG01에서 Wazuh 구성 시 Memory 부족이 발생하는 경우:

`4GB → 6GB`

DB01에서 Database 테스트 시 성능 문제가 발생하는 경우:

`3GB → 4GB`

단, Resource 변경 시 변경 이유와 변경 전후 사양을 기록한다.

예:

DB01

Before:

RAM `3GB`

Issue:

Database Response Delay

Action:

RAM `4GB`로 증설

Result:

Database Test 정상


## 30. Final VM Configuration Goal

최종적으로 다음 VM 환경을 구축한다.

| VM | IP | Role |
|---|---|---|
| DC01 | 10.10.20.10 | AD / DNS |
| DC02 | 10.10.20.11 | AD / DNS |
| DHCP01 | 10.10.20.12 | DHCP |
| FILE01 | 10.10.20.20 | File Server |
| BACKUP01 | 10.10.20.30 | Backup |
| MON01 | 10.10.20.40 | Monitoring |
| LOG01 | 10.10.20.50 | Logging |
| WEB01 | 10.10.20.60 | Web |
| DB01 | 10.10.20.70 | Database |
| MAIL01 | 10.10.20.80 | Mail |
| CLIENT01 | DHCP | Client |

모든 VM은 동일한 Naming Convention과 Network Address Plan을 적용한다.

Host PC의 Resource 제한을 고려하여 필요한 VM만 선택적으로 실행하며, 실제 구축 과정에서 성능을 확인하면서 RAM과 Disk를 조정한다.

VM 생성 완료 후 각 VM의 Network Configuration을 수행하고 이후 Server Role 설치 단계로 진행한다.
