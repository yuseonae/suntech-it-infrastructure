# 04. VMware Network Lab

## 1. Purpose

SunTech IT Infrastructure Lab은 VMware Workstation을 이용하여 실제 제조기업의 네트워크 환경을 가상화하여 구축한다.

실제 기업 환경에서는 Firewall, Core Switch, Access Switch 등의 물리 장비를 사용하지만, 본 프로젝트에서는 VMware의 Virtual Network와 pfSense를 활용하여 유사한 네트워크 구조를 구현한다.

본 단계의 목표는 다음과 같다.

1. 네트워크 영역별 분리
2. Firewall 기반 네트워크 통신 제어
3. Server 및 User Network 구성
4. 네트워크 간 통신 테스트
5. 실제 기업 네트워크 구조에 대한 이해


## 2. Virtual Network Architecture

전체 VMware 네트워크 구조는 다음과 같다.

Internet
↓
pfSense Firewall
↓
VMware Virtual Networks
↓
HQ User / Server / IT Management / Factory / Guest


## 3. VMnet Design

각 업무 영역은 VMware의 독립적인 Virtual Network로 구성한다.

| VMnet | Network | Purpose | Gateway |
|---|---|---|---|
| VMnet10 | 10.10.10.0/24 | HQ User | 10.10.10.1 |
| VMnet20 | 10.10.20.0/24 | Server | 10.10.20.1 |
| VMnet30 | 10.10.30.0/24 | IT Management | 10.10.30.1 |
| VMnet40 | 10.10.40.0/24 | Factory | 10.10.40.1 |
| VMnet50 | 10.10.50.0/24 | Guest | 10.10.50.1 |


## 4. pfSense Firewall

pfSense를 가상 Firewall 및 Router로 사용한다.

pfSense는 VMware에서 하나의 가상머신으로 구성하며, 각 Virtual Network에 연결되는 가상 NIC를 구성한다.

### Network Interfaces

| Interface | Connected Network | IP Address | Role |
|---|---|---|---|
| WAN | Internet / NAT | DHCP | External Network |
| LAN10 | VMnet10 | 10.10.10.1/24 | HQ User Gateway |
| LAN20 | VMnet20 | 10.10.20.1/24 | Server Gateway |
| LAN30 | VMnet30 | 10.10.30.1/24 | IT Management Gateway |
| LAN40 | VMnet40 | 10.10.40.1/24 | Factory Gateway |
| LAN50 | VMnet50 | 10.10.50.1/24 | Guest Gateway |


## 5. Network Connection Structure

VMware 환경의 전체적인 연결 구조는 다음과 같다.

Host PC
→ VMware Workstation
→ pfSense VM
→ VMnet Networks
→ Client / Server VMs


### Network Structure

Internet
→ pfSense WAN
→ pfSense Firewall
→ Internal Virtual Networks

Internal Virtual Networks:

- VMnet10: HQ User
- VMnet20: Server
- VMnet30: IT Management
- VMnet40: Factory
- VMnet50: Guest


## 6. Server Network

Server Network은 VMnet20을 사용한다.

Network:

10.10.20.0/24

Gateway:

10.10.20.1

주요 서버:

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

모든 서버는 Server VLAN에 해당하는 VMnet20에 연결한다.


## 7. HQ User Network

HQ User Network은 VMnet10을 사용한다.

Network:

10.10.10.0/24

Gateway:

10.10.10.1

사용자 PC는 DHCP를 통해 IP 주소를 할당받는다.

DHCP 범위:

10.10.10.100 ~ 10.10.10.199


## 8. IT Management Network

IT Management Network은 VMnet30을 사용한다.

Network:

10.10.30.0/24

Gateway:

10.10.30.1

IT 관리자는 별도의 관리용 VM 또는 PC를 사용하여 서버와 네트워크 장비를 관리한다.

관리 네트워크에서 서버에 대한 관리 접근을 허용하고, 일반 사용자 네트워크에서 관리 네트워크로의 접근은 제한한다.


## 9. Factory Network

Factory Network은 VMnet40을 사용한다.

Network:

10.10.40.0/24

Gateway:

10.10.40.1

공장 업무용 PC는 Factory Network에 연결한다.

DHCP 범위:

10.10.40.100 ~ 10.10.40.199

공장 네트워크는 본사 사용자 네트워크와 분리하여 구성한다.


## 10. Guest Network

Guest Network은 VMnet50을 사용한다.

Network:

10.10.50.0/24

Gateway:

10.10.50.1

Guest Network은 인터넷 접근만 허용하고 내부 네트워크에 대한 접근을 제한한다.

Guest Network에서 다음 네트워크로의 직접적인 접근은 차단한다.

- Server Network
- HQ User Network
- IT Management Network
- Factory Network


## 11. DHCP Design

DHCP는 DHCP01 서버에서 제공하는 것을 기본 구성으로 한다.

DHCP Scope은 다음과 같이 구성한다.

### HQ User

Network:

10.10.10.0/24

DHCP Range:

10.10.10.100 ~ 10.10.10.199

Gateway:

10.10.10.1

DNS:

10.10.20.10


### Factory

Network:

10.10.40.0/24

DHCP Range:

10.10.40.100 ~ 10.10.40.199

Gateway:

10.10.40.1

DNS:

10.10.20.10


### IT Management

Network:

10.10.30.0/24

DHCP Range:

10.10.30.100 ~ 10.10.30.149

Gateway:

10.10.30.1

DNS:

10.10.20.10


### Guest

Guest Network의 DHCP는 초기 구축에서는 pfSense 또는 별도의 DHCP 환경을 사용하는 방식을 검토한다.

Guest Network은 내부 Active Directory DNS에 직접 의존하지 않도록 구성한다.


## 12. Firewall Policy

pfSense에서는 네트워크 영역별 접근 정책을 적용한다.

| Source | Destination | Policy | Purpose |
|---|---|---|---|
| HQ User | Server | Allow Required Services | 업무 서비스 |
| Factory | Server | Allow Required Services | 업무 서비스 |
| IT Management | Server | Allow Management | 서버 관리 |
| IT Management | Network | Allow Management | 장비 관리 |
| HQ User | IT Management | Deny | 관리망 보호 |
| Factory | IT Management | Deny | 관리망 보호 |
| Guest | Internet | Allow | 인터넷 사용 |
| Guest | Internal Network | Deny | 내부망 보호 |

모든 네트워크 통신을 무조건 허용하지 않고 업무에 필요한 서비스와 포트만 단계적으로 허용한다.


## 13. Network Testing

네트워크 구축 후 다음 항목을 테스트한다.

### Basic Connectivity

- Client → Gateway
- Server → Gateway
- Factory Client → Gateway
- IT Management Client → Gateway
- Guest Client → Gateway


### Internal Communication

- HQ User → DC01
- HQ User → FILE01
- Factory → DC01
- Factory → Required Server Services
- IT Management → Server Management


### Security Testing

- Guest → Server 차단
- Guest → HQ User 차단
- Guest → Factory 차단
- HQ User → IT Management 차단
- Factory → IT Management 차단


### Internet Testing

- HQ User → Internet
- Factory → Internet
- IT Management → Internet
- Guest → Internet


## 14. Network Troubleshooting

네트워크 문제가 발생할 경우 다음 순서로 확인한다.

1. VM Network Adapter 연결 상태 확인
2. VMnet 설정 확인
3. IP Address 확인
4. Subnet Mask 확인
5. Default Gateway 확인
6. DNS 설정 확인
7. pfSense Interface 상태 확인
8. Firewall Rule 확인
9. Routing 상태 확인
10. 대상 서버의 서비스 상태 확인

문제 해결 과정은 별도의 Troubleshooting 문서에 기록한다.


## 15. VMware Network Adapter Configuration

각 가상머신의 Network Adapter는 역할에 맞는 VMnet에 연결한다.

### Example

DC01:

VMnet20

FILE01:

VMnet20

WEB01:

VMnet20

DB01:

VMnet20

MAIL01:

VMnet20

HQ Client:

VMnet10

Factory Client:

VMnet40

IT Admin Client:

VMnet30

Guest Client:

VMnet50


## 16. VLAN and VMnet Relationship

실제 기업 환경에서는 물리 스위치의 VLAN을 이용하여 네트워크를 분리한다.

본 프로젝트에서는 VMware Workstation의 독립적인 VMnet을 이용하여 각 네트워크 영역을 분리한다.

따라서 본 환경은 실제 802.1Q VLAN을 직접 구현한 것이 아니라, VLAN으로 분리된 환경을 가상 네트워크로 재현한 Lab 환경이다.

실제 VLAN 환경과 VMware VMnet 환경의 차이를 이해하고 설계 문서에 명확하게 구분하여 기록한다.


## 17. Final Virtual Network Structure

최종 VMware 네트워크 구조는 다음과 같다.

Internet

↓

pfSense Firewall

↓

+-------------------+-------------------+-------------------+-------------------+-------------------+

|                   |                   |                   |                   |

VMnet10             VMnet20             VMnet30             VMnet40             VMnet50

HQ User             Server              IT Management       Factory             Guest

10.10.10.0/24       10.10.20.0/24       10.10.30.0/24       10.10.40.0/24       10.10.50.0/24

|                   |                   |                   |                   |

User PCs             Servers             IT Admin            Factory PCs         Guest Devices


## 18. Design Principles

VMware Network Lab은 다음 원칙을 기준으로 구축한다.

1. 네트워크 영역별 분리
2. Firewall 기반 접근 제어
3. Server Network 보호
4. Management Network 분리
5. Guest Network 격리
6. 고정 IP 기반 서버 관리
7. DHCP 기반 사용자 IP 관리
8. DNS 기반 내부 이름 해석
9. 단계적인 네트워크 테스트
10. 실제 기업 네트워크 구조와 가상 환경의 차이 명확화
