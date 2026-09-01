# 05. Network Implementation

## 1. Purpose

본 문서는 SunTech IT Infrastructure의 네트워크 설계를 실제 VMware 가상 환경에 적용하기 위한 구현 기준을 정의한다.

기존 문서에서 정의한 Network Design, IP Address Plan, Network Topology 및 VMware Network Lab 구성을 바탕으로 실제 서버와 Client VM에 네트워크 설정을 적용한다.

본 문서는 이후 모든 서버 구축 과정에서 Network Configuration과 Connectivity Test의 기준으로 사용한다.


## 2. Implementation Scope

이번 단계에서는 다음 항목을 실제 환경에 적용한다.

- VMware Virtual Network
- Server Network
- Client Network
- Static IP
- DHCP
- DNS
- Gateway
- Hostname Resolution
- Server-to-Server Connectivity
- Client-to-Server Connectivity

네트워크 구축 후 각 항목에 대한 기본 연결 테스트를 수행한다.


## 3. Network Address

전체 SunTech Private Network:

`10.10.0.0/16`

현재 실제 사용하는 Network:

| Network | Address | Purpose |
|---|---|---|
| Client Network | 10.10.10.0/24 | User PC / Client |
| Server Network | 10.10.20.0/24 | Server Infrastructure |

현재 프로젝트에서는 위 두 Network를 중심으로 구축한다.


## 4. Client Network

Network:

`10.10.10.0/24`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.10.1`

DHCP Range:

`10.10.10.100 - 10.10.10.200`

DNS:

- 10.10.20.10
- 10.10.20.11

Client PC는 DHCP를 통해 IP Address를 자동으로 할당받는다.


## 5. Server Network

Network:

`10.10.20.0/24`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

Server는 기본적으로 Static IP를 사용한다.

DNS:

- Primary: 10.10.20.10
- Secondary: 10.10.20.11


## 6. Server Address Mapping

| Hostname | Role | IP |
|---|---|---|
| DC01 | AD / DNS | 10.10.20.10 |
| DC02 | AD / DNS | 10.10.20.11 |
| DHCP01 | DHCP | 10.10.20.12 |
| FILE01 | File Server | 10.10.20.20 |
| BACKUP01 | Backup | 10.10.20.30 |
| MON01 | Monitoring | 10.10.20.40 |
| LOG01 | Logging | 10.10.20.50 |
| WEB01 | Web Server | 10.10.20.60 |
| DB01 | Database | 10.10.20.70 |
| MAIL01 | Mail Server | 10.10.20.80 |


## 7. Network Configuration Principle

### Server

Server는 Static IP를 사용한다.

이유:

- IP 변경 방지
- DNS 등록 안정성
- 서비스 연결 안정성
- Monitoring 설정 편의성
- 장애 분석 편의성

### Client

Client는 DHCP를 사용한다.

이유:

- 사용자 PC의 자동 IP 할당
- 중앙 관리
- IP 충돌 방지
- Gateway / DNS 자동 배포

## 8. Implementation Order

네트워크 구현은 다음 순서로 진행한다.

1. VMware Virtual Network 확인
2. DC01 Network Configuration
3. DC02 Network Configuration
4. DHCP01 Network Configuration
5. FILE01 Network Configuration
6. BACKUP01 Network Configuration
7. MON01 Network Configuration
8. LOG01 Network Configuration
9. WEB01 Network Configuration
10. DB01 Network Configuration
11. MAIL01 Network Configuration
12. Client VM Network Configuration
13. Connectivity Test
14. DNS Test
15. DHCP Test


## 9. VMware Network Configuration

VMware Workstation Pro에서 VM Network Adapter가 올바른 Virtual Network에 연결되어 있는지 확인한다.

현재 프로젝트에서는 VMware의 Virtual Network를 이용하여 내부 Infrastructure Network를 구성한다.

인터넷이 필요한 VM은 NAT Network를 이용할 수 있으며, 내부 Infrastructure 통신은 프로젝트에서 정의한 내부 Network 구성을 기준으로 연결한다.

VM별 Network Adapter 설정은 실제 구축 환경에서 확인 후 기록한다.


## 10. DC01 Network Configuration

DC01:

IP:

`10.10.20.10`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

DNS:

`10.10.20.10`

DC01은 초기 Domain Controller 및 DNS 구축의 기준 서버로 사용한다.

Active Directory와 DNS 구축 이후 필요에 따라 DNS 설정을 보완한다.


## 11. DC02 Network Configuration

DC02:

IP:

`10.10.20.11`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

DNS:

`10.10.20.10`

DC02는 DC01 구축 이후 Domain에 추가하고 Additional Domain Controller로 구성한다.

구축 완료 후 DNS와 Active Directory Replication을 확인한다.


## 12. DHCP01 Network Configuration

DHCP01:

IP:

`10.10.20.12`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

DNS:

- 10.10.20.10
- 10.10.20.11

DHCP01에서 Client Network에 대한 DHCP Scope를 구성한다.


## 13. DHCP Scope

DHCP Scope:

`10.10.10.100 - 10.10.10.200`

Subnet:

`255.255.255.0`

Default Gateway:

`10.10.10.1`

DNS:

- 10.10.20.10
- 10.10.20.11

Domain Name:

`suntech.local`

DHCP 설정 완료 후 Client VM에서 자동으로 IP가 할당되는지 확인한다.


## 14. Static IP Configuration

각 Server VM에 다음 정보를 직접 입력한다.

- IP Address
- Subnet Mask
- Default Gateway
- Preferred DNS
- Alternate DNS

설정 후 `ipconfig /all` 또는 Linux 환경의 `ip addr` 등을 이용하여 실제 설정값을 확인한다.

## 15. Basic Connectivity Test

각 서버의 Network Configuration 완료 후 기본 통신을 확인한다.

### Gateway Test

각 서버에서 Gateway로 Ping을 수행한다.

예:

`ping 10.10.20.1`

정상적으로 응답하는지 확인한다.


### Server Test

DC01을 기준으로 주요 서버의 연결을 확인한다.

예:

`ping 10.10.20.11`

`ping 10.10.20.12`

`ping 10.10.20.20`

`ping 10.10.20.30`

`ping 10.10.20.40`

`ping 10.10.20.50`

`ping 10.10.20.60`

`ping 10.10.20.70`

`ping 10.10.20.80`


## 16. DNS Resolution Test

DNS 구축 이후 Hostname Resolution을 확인한다.

예:

`nslookup dc01.suntech.local`

`nslookup dc02.suntech.local`

`nslookup file01.suntech.local`

`nslookup web01.suntech.local`

`nslookup db01.suntech.local`

`nslookup mail01.suntech.local`

각 Hostname이 올바른 IP Address로 변환되는지 확인한다.


## 17. Client DHCP Test

Client VM의 Network Adapter를 DHCP로 설정한다.

Windows Client에서 다음 명령을 실행한다.

`ipconfig /release`

`ipconfig /renew`

이후 다음 정보를 확인한다.

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server


예상 IP 범위:

`10.10.10.100 - 10.10.10.200`


## 18. Client Connectivity Test

Client에서 Gateway와 주요 Server에 대한 통신을 확인한다.

### Gateway

`ping 10.10.10.1`

### Domain Controller

`ping 10.10.20.10`

`ping 10.10.20.11`

### File Server

`ping 10.10.20.20`

### Web Server

`ping 10.10.20.60`

### Database Server

`ping 10.10.20.70`


## 19. Domain Resolution Test

Client에서 Domain Name을 확인한다.

`nslookup suntech.local`

또한 Domain Controller의 이름을 확인한다.

`nslookup dc01.suntech.local`

정상적으로 IP Address가 반환되는지 확인한다.


## 20. Domain Join Preparation

Client가 Domain에 Join하기 전에 다음 조건을 확인한다.

- Client IP 정상
- Gateway 정상
- DNS 정상
- DC01 통신 가능
- DC02 통신 가능
- `suntech.local` Resolution 정상

특히 Client의 DNS가 외부 DNS가 아닌 Active Directory DNS를 사용하고 있는지 확인한다.


## 21. Port Connectivity Test

서비스 구축 이후 필요한 Port의 연결 여부를 확인한다.

예:

Web:

`443/TCP`

Database:

`5432/TCP`

Mail:

`25/TCP`

`587/TCP`

`993/TCP`

Monitoring:

`10050/TCP`

필요한 서비스 Port만 허용하고 불필요한 Port는 차단하는 것을 원칙으로 한다.


## 22. Network Troubleshooting

네트워크 문제가 발생할 경우 다음 순서로 확인한다.

1. VMware Network Adapter
2. Virtual Network
3. IP Address
4. Subnet Mask
5. Gateway
6. DNS
7. Firewall
8. Service
9. Port
10. Application

문제 해결 과정은 별도의 Troubleshooting 문서에 기록한다.

## 23. Troubleshooting Example

### Case 1. Client에서 Domain을 찾지 못하는 경우

확인 순서:

Client IP
→ DNS Address
→ DC01 Ping
→ DNS Service
→ `nslookup suntech.local`
→ Active Directory DNS


### Case 2. WEB01에서 DB01 연결 실패

확인 순서:

WEB01 IP
→ DB01 IP
→ DNS Resolution
→ DB01 Ping
→ TCP 5432
→ PostgreSQL Service
→ Database Permission


### Case 3. Client에서 FILE01 접근 실패

확인 순서:

Client IP
→ DNS
→ FILE01 Resolution
→ FILE01 Ping
→ SMB 445
→ Share Permission
→ NTFS Permission


## 24. Network Security

네트워크는 필요한 통신만 허용하는 것을 기본 원칙으로 한다.

기본 원칙:

`Default Deny`

필요한 서비스:

`Explicit Allow`

예:

WEB01에서 DB01의 PostgreSQL Database에 접근하는 경우:

`WEB01 → TCP 5432 → DB01`

필요한 통신만 허용하고 불필요한 접근은 차단한다.


## 25. Network Segmentation

현재 프로젝트에서는 다음 두 Network를 논리적으로 구분한다.

### Client

`10.10.10.0/24`

### Server

`10.10.20.0/24`

실제 기업 환경에서는 VLAN과 Layer 3 Routing을 통해 더욱 세분화할 수 있다.

이번 프로젝트에서는 VMware 실습 환경의 규모를 고려하여 Client와 Server를 우선 분리하고 이후 필요에 따라 추가 Network Segment를 검토한다.


## 26. Implementation Evidence

실제 구축 과정에서 주요 설정과 테스트 결과를 캡처한다.

### VMware

- Virtual Network 설정
- VM Network Adapter

### Server

- IP Configuration
- DNS Configuration
- Hostname

### DHCP

- Scope
- Address Pool
- Lease
- DHCP Options

### DNS

- Forward Lookup Zone
- Reverse Lookup Zone
- Host Record

### Connectivity

- Ping
- nslookup
- Port Test


## 27. Network Validation Checklist

### VMware

- [ ] Virtual Network 확인
- [ ] VM Network Adapter 확인
- [ ] Network 연결 확인

### Server

- [ ] DC01 IP 설정
- [ ] DC02 IP 설정
- [ ] DHCP01 IP 설정
- [ ] FILE01 IP 설정
- [ ] BACKUP01 IP 설정
- [ ] MON01 IP 설정
- [ ] LOG01 IP 설정
- [ ] WEB01 IP 설정
- [ ] DB01 IP 설정
- [ ] MAIL01 IP 설정

### DHCP

- [ ] DHCP Scope 생성
- [ ] IP Address Assignment
- [ ] Gateway Assignment
- [ ] DNS Assignment
- [ ] Domain Name Assignment

### DNS

- [ ] Forward Lookup
- [ ] Reverse Lookup
- [ ] Server Resolution
- [ ] Domain Resolution

### Connectivity

- [ ] Gateway Ping
- [ ] Server Ping
- [ ] Client → Server Ping
- [ ] DNS Resolution
- [ ] Port Test


## 28. Final Network Implementation

SunTech Network는 다음 기준으로 실제 구축한다.

Client Network:

`10.10.10.0/24`

Server Network:

`10.10.20.0/24`

Client는 DHCP01을 통해 IP를 자동으로 할당받으며 Server는 Static IP를 사용한다.

DC01과 DC02는 Active Directory DNS를 제공한다.

모든 서버는 역할에 맞는 Static IP를 사용하고 DNS를 통해 Hostname으로 접근할 수 있도록 구성한다.

실제 구축 후 Ping, DNS Resolution, DHCP Assignment 및 Service Port Test를 수행하여 네트워크 정상 여부를 검증한다.


## 29. Implementation Completion Criteria

Network Implementation은 다음 조건을 만족하면 완료로 판단한다.

1. VMware Network 정상
2. Server Static IP 정상
3. Client DHCP 정상
4. Gateway 통신 정상
5. Server 간 통신 정상
6. Client → Server 통신 정상
7. DNS Resolution 정상
8. Domain Resolution 정상
9. 필요한 Service Port 통신 정상
10. 주요 설정 화면 및 테스트 결과 캡처 완료


## 30. Status

Current Status:

`PLANNED`

실제 구축이 시작되면 단계별 상태를 다음과 같이 변경한다.

`PLANNED`
→ `IN PROGRESS`
→ `TESTING`
→ `COMPLETED`

문제가 발생한 경우:

`ISSUE`

로 기록하고 Troubleshooting 과정을 별도로 남긴다.
