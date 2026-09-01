# 03. Network Topology

## 1. Topology Overview

SunTech의 네트워크는 Internet, Firewall, Core Switch를 중심으로 구성하며, 업무 목적에 따라 VLAN을 분리한다.

본사 사용자, 서버, IT 관리, 공장 및 Guest 네트워크를 논리적으로 분리하여 네트워크 보안성과 관리 효율성을 확보한다.

현재 단계에서는 기본적인 네트워크 토폴로지를 구성하고, 향후 서버 및 서비스 설계가 완료되면 서버 구성을 최종 반영한다.


## 2. Physical Network Structure

전체적인 물리 네트워크 구조는 다음과 같다.

```text
                                Internet
                                    |
                             [ Firewall / Router ]
                                    |
                              [ Core Switch ]
                                    |
       +----------------------------+----------------------------+
       |              |             |             |              |
    VLAN 10        VLAN 20       VLAN 30       VLAN 40        VLAN 50
    HQ User         Server       IT Mgmt        Factory         Guest
       |              |             |             |              |
[HQ Access SW]    [Server]     [IT Admin PC] [Factory SW]      Wi-Fi
       |              |                            |
    직원 PC       +----+----+                  공장 PC
                  |    |    |
                DC01 DC02 FILE01
                         |
                      BACKUP01
```


## 3. Internet and Firewall

외부 인터넷과 내부 네트워크 사이에는 Firewall을 배치한다.

Firewall은 외부 및 내부 네트워크 간의 트래픽을 제어하며, 다음과 같은 기능을 수행한다.

- Internet Access Control
- NAT
- 외부 접근 제어
- 내부 네트워크 접근 제어
- Guest Network 인터넷 접근 제어
- 보안 이벤트 및 트래픽 로그 관리

외부에서 내부 서버로의 직접적인 접근은 기본적으로 차단하며, 필요한 서비스에 대해서만 예외적으로 허용한다.


## 4. Core Switch

Core Switch는 내부 네트워크의 중심 장비로 사용한다.

각 VLAN을 연결하고 VLAN 간 통신이 필요한 경우 적절한 라우팅 및 접근 제어 정책을 적용한다.

```text
                    [ Core Switch ]
                         |
        +----------------+----------------+
        |                |                |
     VLAN 10          VLAN 20          VLAN 30
     HQ User           Server          IT Mgmt
        |
     VLAN 40
     Factory
        |
     VLAN 50
     Guest
```

실제 환경에서는 Core Switch의 역할과 Firewall의 역할을 명확히 구분하고, 보안 정책은 Firewall 또는 Layer 3 장비의 설계에 따라 적용한다.


## 5. HQ User Network

본사 사용자는 VLAN 10을 사용한다.

```text
VLAN 10 - HQ User
        |
  [HQ Access Switch]
        |
   +----+----+----+
   |    |    |    |
  PC   PC   PC   PC
```

사용자 PC는 DHCP를 통해 IP 주소를 할당받는다.

부서별 파일 접근 권한은 네트워크를 통해 구분하지 않고 Active Directory Group과 File Server 권한을 통해 관리한다.


## 6. Server Network

서버는 VLAN 20에 배치한다.

```text
VLAN 20 - Server
       |
       +── DC01
       +── DC02
       +── FILE01
       +── BACKUP01
       +── Additional Servers
```

서버에는 고정 IP 주소를 사용한다.

메일 서버, 모니터링 서버, 로그 서버, 웹 서버, DB 서버 등의 추가 서버는 Server & Service Design 단계에서 필요성과 역할을 검토한 후 최종적으로 추가한다.


## 7. IT Management Network

IT 관리자는 VLAN 30을 사용한다.

```text
VLAN 30 - IT Management
          |
     IT Admin PC
          |
     +----+----+
     |         |
   Server    Network
   Mgmt      Device Mgmt
```

IT Management 네트워크는 일반 사용자의 접근을 제한한다.

IT 관리자는 해당 네트워크를 통해 서버 및 네트워크 장비의 관리 작업을 수행한다.

예시:

- RDP
- PowerShell Remoting
- SSH
- Web Management Console
- Monitoring Console


## 8. Factory Network

생산공장의 업무용 PC 및 관련 시스템은 VLAN 40으로 분리한다.

```text
VLAN 40 - Factory
        |
[Factory Access Switch]
        |
   +----+----+----+
   |    |    |    |
  PC   PC   PC   PC
```

공장 네트워크는 본사 사용자 네트워크와 논리적으로 분리한다.

공장 사용자가 업무에 필요한 서버 서비스에는 접근할 수 있도록 허용하되, 불필요한 내부 네트워크 접근은 제한한다.


## 9. Guest Network

방문객 및 외부 협력업체를 위한 네트워크는 VLAN 50으로 분리한다.

```text
Guest Device
     |
  VLAN 50
     |
 [ Firewall ]
     |
  Internet
```

Guest 네트워크는 인터넷 접근만 허용하고 다음과 같은 내부 네트워크 접근은 차단한다.

```text
Guest
  |
  +----> Internet       Allow
  |
  X----> Server         Deny
  X----> HQ User        Deny
  X----> Factory        Deny
  X----> IT Mgmt        Deny
```


## 10. VLAN Summary

| VLAN ID | Network | Purpose | Main Devices |
|---:|---|---|---|
| 10 | `10.10.10.0/24` | HQ User | Employee PCs |
| 20 | `10.10.20.0/24` | Server | Infrastructure Servers |
| 30 | `10.10.30.0/24` | IT Management | IT Admin PCs |
| 40 | `10.10.40.0/24` | Factory | Factory PCs |
| 50 | `10.10.50.0/24` | Guest | Guest Devices |


## 11. Security Zones

네트워크 영역은 다음과 같이 구분한다.

### User Zone

```text
VLAN 10 - HQ User
VLAN 40 - Factory
```

일반적인 업무용 사용자 환경이다.


### Server Zone

```text
VLAN 20 - Server
```

기업의 핵심 IT 서비스를 제공하는 서버 환경이다.


### Management Zone

```text
VLAN 30 - IT Management
```

IT 관리자가 시스템과 네트워크 장비를 관리하기 위한 환경이다.


### Guest Zone

```text
VLAN 50 - Guest
```

외부 방문객 및 협력업체가 인터넷을 이용하기 위한 별도 환경이다.


## 12. Inter-VLAN Communication

VLAN 간 통신은 업무상 필요한 경우에만 허용한다.

| Source | Destination | Default Policy | Example |
|---|---|---|---|
| HQ User | Server | Allow Required Services | AD, DNS, File Server |
| Factory | Server | Allow Required Services | AD, DNS, 업무 서비스 |
| IT Management | Server | Allow Management Services | RDP, PowerShell |
| IT Management | Network Devices | Allow Management | SSH, HTTPS |
| HQ User | IT Management | Deny | Management Network Protection |
| Factory | IT Management | Deny | Management Network Protection |
| HQ User | Factory | Deny by Default | Unnecessary Direct Access |
| Factory | HQ User | Deny by Default | Unnecessary Direct Access |
| Guest | Internet | Allow | Internet Access |
| Guest | Internal Network | Deny | Internal Network Protection |

`Allow` 정책은 모든 포트와 프로토콜을 허용한다는 의미가 아니라, 실제 운영에 필요한 서비스와 포트만 허용하는 것을 의미한다.


## 13. Future Expansion

현재 네트워크는 기본적인 기업 IT 환경을 기준으로 구성한다.

향후 다음과 같은 영역을 추가할 수 있다.

- Wireless Network
- Production Equipment Network
- IoT Network
- Security Management Network
- Additional Server Network
- DMZ
- VPN Network

추가 네트워크가 필요한 경우 기존 IP Address Plan과 VLAN 구조를 함께 검토하여 확장한다.


## 14. Design Principles

SunTech 네트워크는 다음 원칙을 기준으로 설계한다.

1. Network Segmentation
2. Least Privilege
3. Defense in Depth
4. Management Network Isolation
5. Guest Network Isolation
6. Server Network Protection
7. Expandable IP Address Design
8. Required-Service-Based Communication

이를 통해 사용자 편의성과 보안성을 동시에 확보할 수 있는 제조기업 IT 네트워크를 구축하는 것을 목표로 한다.
