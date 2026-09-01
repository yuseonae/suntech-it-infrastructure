# 01. Network Design

## 1. Network Overview

SunTech는 약 200명의 임직원이 근무하는 제조기업으로, 본사와 생산공장으로 구성된 IT 환경을 가정한다.

네트워크는 사용자, 서버, IT 관리, 공장 및 방문객 네트워크를 논리적으로 분리하여 보안성과 관리 효율성을 확보하도록 설계한다.

전체 내부 네트워크는 `10.10.0.0/16` 사설 IP 주소 공간을 사용하며, 각 네트워크 영역은 VLAN 단위로 분리한다.


## 2. Network Design Objectives

본 네트워크 설계의 목표는 다음과 같다.

1. 업무 영역별 네트워크를 논리적으로 분리한다.
2. 주요 서버를 사용자 네트워크와 분리하여 보호한다.
3. 본사와 공장 네트워크를 분리한다.
4. IT 관리용 네트워크를 별도로 구성한다.
5. 방문객 및 외부 사용자의 내부망 접근을 제한한다.
6. 네트워크 간 통신은 필요한 서비스만 허용한다.
7. 향후 사용자 및 시스템 증가에 대응할 수 있도록 확장 가능한 IP 구조를 설계한다.


## 3. Network Architecture

전체적인 네트워크 구조는 다음과 같다.

```text
                         Internet
                             |
                     [ Firewall / Router ]
                             |
                        [ Core Switch ]
                             |
       +---------------------+---------------------+
       |           |           |           |        |
    VLAN 10     VLAN 20     VLAN 30     VLAN 40  VLAN 50
    HQ User      Server     IT Mgmt      Factory   Guest
       |           |           |           |        |
    직원 PC      서버들     IT 관리자    공장 PC   방문객
```


## 4. VLAN Design

### VLAN 10 - HQ User

본사 임직원이 사용하는 업무용 네트워크이다.

대상:

- Management
- Sales
- IT
- 기타 본사 업무용 PC

부서별 파일 접근 권한은 네트워크가 아닌 Active Directory Group 및 File Server 권한을 통해 관리한다.


### VLAN 20 - Server

기업의 주요 IT 인프라 서버를 배치하는 네트워크이다.

대상:

- Domain Controller
- DNS
- DHCP
- File Server
- Backup Server
- 향후 추가되는 인프라 서버

일반 사용자 네트워크에서 서버로의 접근은 필요한 서비스와 포트만 허용한다.


### VLAN 30 - IT Management

IT 담당자가 서버 및 네트워크 장비를 관리하기 위한 전용 네트워크이다.

대상:

- IT Administrator PC
- 서버 관리 환경
- 네트워크 장비 관리 환경

일반 사용자 및 공장 네트워크에서 IT Management 네트워크로의 직접 접근은 기본적으로 차단한다.


### VLAN 40 - Factory

생산공장의 업무용 PC 및 생산환경을 위한 네트워크이다.

대상:

- Production
- Quality
- R&D
- 생산 관련 업무 시스템 및 장비

공장 네트워크는 일반 본사 사용자 네트워크와 분리하여 관리한다.


### VLAN 50 - Guest

방문객 및 외부 협력업체 등이 사용할 수 있는 네트워크이다.

Guest 네트워크에서는 인터넷 사용만 허용하고 사내 내부 네트워크에 대한 접근은 차단한다.


## 5. Inter-VLAN Communication Policy

네트워크 간 통신은 기본적으로 최소 권한 원칙을 적용한다.

| Source | Destination | Policy | Purpose |
|---|---|---|---|
| HQ User | Server | Allow | AD, DNS, File Server 등 업무 서비스 |
| Factory | Server | Allow | AD, DNS 및 필요한 업무 서비스 |
| IT Management | Server | Allow | 서버 관리 |
| IT Management | Network Devices | Allow | 네트워크 장비 관리 |
| HQ User | IT Management | Deny | 관리망 보호 |
| Factory | IT Management | Deny | 관리망 보호 |
| HQ User | Factory | Deny by default | 불필요한 직접 통신 제한 |
| Factory | HQ User | Deny by default | 불필요한 직접 통신 제한 |
| Guest | Internet | Allow | 인터넷 사용 |
| Guest | Internal Network | Deny | 내부망 보호 |

`Allow`는 모든 통신을 허용한다는 의미가 아니라, 업무에 필요한 서비스와 포트만 허용하는 것을 의미한다.


## 6. Network Security Principles

### 6.1 Least Privilege

사용자와 시스템은 업무 수행에 필요한 최소한의 네트워크 접근 권한만 부여한다.


### 6.2 Network Segmentation

사용자, 서버, 관리 및 공장 환경을 VLAN으로 분리하여 하나의 네트워크에서 발생한 문제가 다른 영역으로 확산되는 것을 최소화한다.


### 6.3 Server Protection

서버 VLAN은 일반 사용자 VLAN과 분리하며, 서버에 대한 접근은 필요한 서비스 단위로 제한한다.


### 6.4 Management Network Protection

IT Management VLAN은 일반 사용자에게 노출하지 않으며, IT 관리자만 접근할 수 있도록 구성한다.


### 6.5 Guest Network Isolation

Guest VLAN은 내부 네트워크와 분리하고 인터넷 접근만 허용한다.


## 7. Network Expansion

현재 설계에서는 다음과 같은 VLAN을 사용한다.

```text
VLAN 10  - HQ User
VLAN 20  - Server
VLAN 30  - IT Management
VLAN 40  - Factory
VLAN 50  - Guest
```

향후 필요에 따라 다음과 같은 네트워크를 추가할 수 있다.

- 추가 서버 네트워크
- 보안 장비 네트워크
- 무선 네트워크
- 별도의 생산설비 네트워크
- IoT 및 장비 관리 네트워크

전체 사설 IP 주소 공간으로 `10.10.0.0/16`을 확보하고 VLAN별 `/24` 네트워크를 할당하여 향후 확장에 대응한다.
