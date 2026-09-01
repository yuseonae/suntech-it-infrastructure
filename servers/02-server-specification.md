# 02. Server Specification

## 1. Host Environment

SunTech IT Infrastructure Lab은 개인 PC의 VMware Workstation 환경에서 구축한다.

### Host PC Specification

| Item | Specification |
|---|---|
| CPU | Intel Core i7-9700F |
| CPU Cores | 8 Cores / 8 Threads |
| RAM | 32GB |
| System Drive | C: 약 465GB |
| VM Storage | D: 약 1.81TB |
| Available VM Storage | 약 1.29TB |

VMware 가상머신은 저장공간과 관리 편의성을 고려하여 D: 드라이브에 저장한다.


## 2. VMware Storage Structure

가상머신은 다음과 같은 디렉터리 구조로 관리한다.

D:\VMware\SunTech\

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

VM별로 별도의 디렉터리를 사용하여 가상 디스크와 VMware 설정 파일을 관리한다.


## 3. Virtual Machine Specification

| Hostname | OS | vCPU | RAM | Disk | IP Address | VLAN |
|---|---|---:|---:|---:|---|---:|
| DC01 | Windows Server | 2 | 3GB | 60GB | 10.10.20.10 | 20 |
| DC02 | Windows Server | 2 | 3GB | 60GB | 10.10.20.11 | 20 |
| DHCP01 | Windows Server | 1 | 2GB | 40GB | 10.10.20.12 | 20 |
| FILE01 | Windows Server | 2 | 4GB | 100GB | 10.10.20.20 | 20 |
| BACKUP01 | Windows Server | 2 | 3GB | 100GB | 10.10.20.30 | 20 |
| MON01 | Linux | 2 | 2GB | 40GB | 10.10.20.40 | 20 |
| LOG01 | Linux | 2 | 2GB | 50GB | 10.10.20.50 | 20 |
| WEB01 | Linux | 2 | 2GB | 40GB | 10.10.20.60 | 20 |
| DB01 | Linux | 2 | 2GB | 50GB | 10.10.20.70 | 20 |
| MAIL01 | Linux | 2 | 2GB | 50GB | 10.10.20.80 | 20 |


## 4. Windows Server Systems

다음 서버는 Windows Server 기반으로 구성한다.

### DC01

- AD DS
- DNS
- Primary Domain Controller

### DC02

- AD DS
- DNS
- Additional Domain Controller
- Active Directory Replication

### DHCP01

- DHCP Server
- DHCP Scope Management

### FILE01

- File Server
- SMB
- NTFS Permission
- Share Permission
- FSRM

### BACKUP01

- Windows Server Backup
- System State Backup
- File Server Backup


## 5. Linux Server Systems

다음 서버는 Linux 기반으로 구성한다.

### MON01

Monitoring Server 역할을 수행한다.

주요 모니터링 대상:

- Windows Servers
- Linux Servers
- Network Devices
- CPU
- Memory
- Disk
- Network
- Service Status


### LOG01

Central Log Server 역할을 수행한다.

주요 수집 대상:

- Domain Controller
- File Server
- Web Server
- Database Server
- Mail Server
- Network Devices


### WEB01

내부 웹 서비스를 제공한다.

주요 역할:

- Web Server
- Internal Web Application


### DB01

WEB01에서 사용하는 데이터베이스를 제공한다.

주요 역할:

- Database Server
- Application Data Storage


### MAIL01

기업 내부 메일 서비스를 제공한다.

주요 역할:

- SMTP
- IMAP
- Mail Account Management
- Internal Mail Delivery
- Mail Log


## 6. IP Address Allocation

모든 서버는 Server VLAN에 배치한다.

Server VLAN:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

DNS:

`10.10.20.10`

### Server IP Allocation

| IP Address | Hostname | Role |
|---|---|---|
| 10.10.20.10 | DC01 | AD DS / DNS |
| 10.10.20.11 | DC02 | AD DS / DNS |
| 10.10.20.12 | DHCP01 | DHCP |
| 10.10.20.20 | FILE01 | File Server |
| 10.10.20.30 | BACKUP01 | Backup |
| 10.10.20.40 | MON01 | Monitoring |
| 10.10.20.50 | LOG01 | Central Log |
| 10.10.20.60 | WEB01 | Web Server |
| 10.10.20.70 | DB01 | Database |
| 10.10.20.80 | MAIL01 | Mail Server |


## 7. Resource Allocation Strategy

현재 Host PC는 32GB RAM과 8코어 CPU를 사용한다.

모든 가상머신을 동시에 높은 리소스로 실행하지 않고, 실제 구축 단계에서는 필요한 서버만 실행하여 호스트 PC의 자원을 효율적으로 사용한다.

초기 할당 RAM은 다음과 같다.

- DC01: 3GB
- DC02: 3GB
- DHCP01: 2GB
- FILE01: 4GB
- BACKUP01: 3GB
- MON01: 2GB
- LOG01: 2GB
- WEB01: 2GB
- DB01: 2GB
- MAIL01: 2GB

총 할당 RAM은 약 25GB이다.

실제 운영 시 Host OS에 필요한 메모리를 확보하고, VM의 성능에 따라 RAM을 조정한다.


## 8. VM Operation Strategy

32GB RAM 환경에서 모든 VM을 항상 실행하지 않는다.

### Core Infrastructure

평상시 우선적으로 실행할 서버:

- DC01
- DC02
- DHCP01
- FILE01

### Additional Services

필요한 테스트 또는 운영 시 실행:

- BACKUP01
- MON01
- LOG01
- WEB01
- DB01
- MAIL01

서비스 구축 단계에 따라 필요한 VM만 실행하여 시스템 자원을 관리한다.


## 9. Server Build Order

가상머신은 다음 순서로 구축한다.

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

핵심 인프라를 먼저 구축하고 정상 동작을 확인한 후 운영 관리 및 업무 서비스를 추가한다.


## 10. OS Selection Principle

Windows 기반 인프라 서비스는 Windows Server를 사용한다.

AD DS, DNS, DHCP, File Server 및 Backup 환경은 Windows Server 기반으로 구성하여 Windows 기업 환경의 시스템 운영 경험을 확보한다.

Monitoring, Logging, Web, Database 및 Mail 서비스는 Linux 기반 서버를 우선 검토한다.

Linux 배포판 및 각 서비스에 사용할 솔루션은 실제 구축 단계에서 난이도와 호환성을 고려하여 최종 선정한다.


## 11. Server Dependency

서버 구축 및 서비스 실행 시 다음 의존관계를 고려한다.

DC01 → DC02

DC01 / DNS → DHCP01

AD / DNS → FILE01

AD / DNS → WEB01

WEB01 → DB01

AD / DNS → MAIL01

모니터링 대상 서버 → MON01

로그 발생 서버 → LOG01

백업 대상 서버 → BACKUP01


## 12. Design Principles

서버 환경은 다음 원칙을 기준으로 설계한다.

1. 역할별 서버 분리
2. 핵심 인프라 이중화
3. 고정 IP 기반 서버 관리
4. VMware 기반 가상화
5. 최소 권한 원칙
6. 백업 및 복구
7. 모니터링
8. 중앙 로그 관리
9. 단계적인 서비스 구축
10. 실제 기업 IT 운영 환경을 고려한 설계


## 13. Implementation Scope

이번 프로젝트에서는 단순히 서버를 설치하는 것에 그치지 않고 각 서버의 역할에 맞는 서비스를 실제로 구축하고 테스트한다.

주요 구현 범위:

- Active Directory
- DNS
- DHCP
- File Server
- FSRM
- Backup
- Monitoring
- Central Log
- Internal Web Service
- Database
- Mail Service

각 서비스는 기본 기능을 먼저 구현한 후 필요에 따라 보안 및 운영 기능을 단계적으로 추가한다.


## 14. Resource Optimization

개인 PC 기반의 가상화 환경이라는 점을 고려하여 고가용성이나 대규모 서버 환경을 그대로 구현하지 않는다.

대신 제한된 CPU와 RAM 환경에서도 기업 IT 인프라의 핵심 구조를 경험할 수 있도록 최소 사양의 가상머신을 구성한다.

필요한 경우 테스트가 끝난 서버의 VM을 종료하거나 리소스를 일시적으로 조정하여 다른 서비스의 구축에 활용한다.
