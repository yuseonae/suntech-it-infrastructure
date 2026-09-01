# 04. DHCP and DNS Design

## 1. Purpose

SunTech IT Infrastructure에서 사용자 및 서버의 네트워크 통신을 안정적으로 운영하기 위해 DHCP와 DNS 환경을 설계한다.

DHCP는 사용자 PC 및 업무용 장비에 IP 주소와 네트워크 설정을 자동으로 할당하고, DNS는 Active Directory 및 내부 서버의 이름 해석을 담당한다.

특히 Active Directory는 DNS에 의존하므로 DNS를 핵심 인프라 서비스로 구성한다.


## 2. Service Architecture

DHCP와 DNS는 다음과 같은 구조로 구성한다.

```text
                    Network
                       |
                 pfSense Firewall
                       |
              Internal Network
                       |
          +------------+------------+
          |                         |
       DHCP01                    DC01 / DC02
       DHCP Server               DNS Server
          |                         |
          |                    Active Directory
          |
    IP Address Assignment
          |
      Client PCs
```

기본적인 역할은 다음과 같이 구분한다.

| Server | Service | Role |
|---|---|---|
| DHCP01 | DHCP Server | Client IP Address Assignment |
| DC01 | DNS Server | Internal DNS / AD DNS |
| DC02 | DNS Server | Secondary DNS / AD DNS |

DC01과 DC02는 모두 Active Directory 통합 DNS를 구성한다.


## 3. DHCP Server

DHCP 서비스는 별도의 서버인 DHCP01에서 제공한다.

### DHCP01

Hostname:

`DHCP01`

IP Address:

`10.10.20.12`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

DHCP01 자체는 고정 IP를 사용한다.


## 4. DHCP Scope Design

사용자 및 업무용 네트워크별로 DHCP Scope을 구분한다.

### HQ User Network

Network:

`10.10.10.0/24`

Gateway:

`10.10.10.1`

DHCP Range:

`10.10.10.100 ~ 10.10.10.199`

Subnet Mask:

`255.255.255.0`

Primary DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

DNS Domain:

`suntech.local`


### IT Management Network

Network:

`10.10.30.0/24`

Gateway:

`10.10.30.1`

DHCP Range:

`10.10.30.100 ~ 10.10.30.149`

Subnet Mask:

`255.255.255.0`

Primary DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

DNS Domain:

`suntech.local`


### Factory Network

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

DHCP Range:

`10.10.40.100 ~ 10.10.40.199`

Subnet Mask:

`255.255.255.0`

Primary DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

DNS Domain:

`suntech.local`


### Guest Network

Network:

`10.10.50.0/24`

Gateway:

`10.10.50.1`

Guest Network의 DHCP는 내부 Active Directory 환경과 분리한다.

Guest Client에는 내부 Active Directory DNS를 직접 제공하지 않으며, 외부 DNS 또는 pfSense에서 제공하는 DNS 서비스를 사용하도록 구성한다.


## 5. DHCP Reservation

서버 및 주요 인프라 장비는 DHCP로 IP를 할당하지 않고 고정 IP를 사용한다.

다음 장비는 Static IP를 사용한다.

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
- pfSense

사용자 PC와 일반 업무용 장비는 DHCP를 통해 IP를 할당받는다.


## 6. IP Address Allocation Principle

네트워크별 IP 주소는 다음 원칙으로 관리한다.

### Infrastructure / Server

`10.10.20.1 ~ 10.10.20.99`

주요 서버에 고정 IP를 할당한다.

### Client DHCP

각 사용자 네트워크에서 `.100 ~ .199` 범위를 DHCP Pool로 사용한다.

### Reserved Area

`.200 ~ .254`는 향후 확장 및 특수 장비를 위한 영역으로 남겨둔다.


## 7. DNS Architecture

Active Directory 환경에서는 DC01과 DC02를 DNS Server로 사용한다.

DNS 구조:

```text
                DNS Client
                    |
          +---------+---------+
          |                   |
        DC01                DC02
     10.10.20.10         10.10.20.11
          |                   |
          +---------+---------+
                    |
             Active Directory
```

DC01과 DC02는 모두 `suntech.local` DNS Zone을 관리한다.


## 8. Internal DNS Zone

Active Directory Domain:

`suntech.local`

DNS Zone:

`suntech.local`

주요 서버의 DNS 이름은 다음과 같이 구성한다.

| Hostname | FQDN | IP Address |
|---|---|---|
| DC01 | dc01.suntech.local | 10.10.20.10 |
| DC02 | dc02.suntech.local | 10.10.20.11 |
| DHCP01 | dhcp01.suntech.local | 10.10.20.12 |
| FILE01 | file01.suntech.local | 10.10.20.20 |
| BACKUP01 | backup01.suntech.local | 10.10.20.30 |
| MON01 | mon01.suntech.local | 10.10.20.40 |
| LOG01 | log01.suntech.local | 10.10.20.50 |
| WEB01 | web01.suntech.local | 10.10.20.60 |
| DB01 | db01.suntech.local | 10.10.20.70 |
| MAIL01 | mail01.suntech.local | 10.10.20.80 |


## 9. DNS Forward Lookup

Forward Lookup Zone은 호스트 이름을 IP 주소로 변환한다.

예:

```text
dc01.suntech.local
        ↓
10.10.20.10
```

```text
file01.suntech.local
        ↓
10.10.20.20
```

```text
web01.suntech.local
        ↓
10.10.20.60
```

이를 통해 사용자는 IP 주소 대신 서버의 Hostname 또는 FQDN을 사용하여 내부 서비스를 이용할 수 있다.


## 10. DNS Reverse Lookup

Reverse Lookup Zone은 IP 주소를 Hostname으로 변환한다.

예:

```text
10.10.20.10
        ↓
dc01.suntech.local
```

Reverse Lookup은 네트워크 장애 분석 및 로그 확인 시 유용하게 활용할 수 있도록 구성한다.


## 11. Active Directory and DNS Integration

Active Directory는 DNS를 이용하여 Domain Controller 및 Active Directory 서비스를 찾는다.

사용자가 `suntech.local` 도메인에 로그인하는 과정에서 DNS를 통해 Domain Controller를 찾을 수 있어야 한다.

기본 흐름:

```text
Client
   |
   ↓
DNS Query
   |
   ↓
DC01 / DC02
   |
   ↓
Active Directory
   |
   ↓
Authentication
```

따라서 Active Directory Client가 외부 DNS만 사용하도록 설정하지 않는다.

도메인에 가입된 Client는 기본적으로 내부 AD DNS를 사용하도록 구성한다.


## 12. DNS Client Configuration

Domain-Joined Client의 DNS 설정은 다음과 같이 구성한다.

Primary DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

DNS Domain:

`suntech.local`

Client는 DC01을 우선 사용하고 DC01에 문제가 발생할 경우 DC02를 사용할 수 있도록 구성한다.


## 13. DHCP and DNS Integration

DHCP Server는 Client에게 다음 네트워크 정보를 전달한다.

- IP Address
- Subnet Mask
- Default Gateway
- Primary DNS
- Secondary DNS
- DNS Domain

예:

```text
Client
   |
   | DHCP Request
   ↓
DHCP01
   |
   +-- IP Address: 10.10.10.100
   +-- Mask: 255.255.255.0
   +-- Gateway: 10.10.10.1
   +-- DNS: 10.10.20.10
   +-- Secondary DNS: 10.10.20.11
   +-- Domain: suntech.local
```

이를 통해 사용자 PC가 자동으로 내부 DNS 및 Active Directory 환경을 사용할 수 있도록 한다.


## 14. DHCP Relay

DHCP Server와 Client가 서로 다른 네트워크에 존재하므로 DHCP Broadcast가 네트워크를 넘어갈 수 있도록 DHCP Relay 또는 이에 준하는 네트워크 구성을 적용한다.

예상 구조:

```text
HQ User Network
10.10.10.0/24
       |
       ↓
pfSense
       |
       ↓
Server Network
10.10.20.0/24
       |
       ↓
DHCP01
10.10.20.12
```

Factory Network과 IT Management Network에서도 동일한 방식으로 DHCP 요청을 DHCP01에 전달한다.

실제 VMware 환경에서 DHCP Relay 구현 방식은 pfSense 설정을 기준으로 구성한다.


## 15. DHCP Lease Management

DHCP Server는 Client에게 일정 기간 동안 IP 주소를 임대한다.

Lease 정보를 통해 다음 사항을 확인할 수 있도록 한다.

- Client IP Address
- MAC Address
- Hostname
- Lease Start Time
- Lease Expiration Time

네트워크 장애 발생 시 DHCP Lease 정보를 활용하여 특정 IP를 사용하는 장비를 추적할 수 있도록 한다.


## 16. DNS Forwarder

내부 DNS에서 확인할 수 없는 외부 도메인은 외부 DNS 또는 pfSense를 통해 조회할 수 있도록 구성한다.

기본적인 흐름:

```text
Internal Client
       |
       ↓
DC01 / DC02
       |
       ↓
External DNS Resolution
       |
       ↓
Internet
```

내부 `suntech.local` 영역은 내부 DNS에서 직접 관리하고 외부 도메인은 Forwarding을 통해 조회한다.


## 17. DNS Security Considerations

DNS 서버에는 다음 원칙을 적용한다.

1. 내부 DNS와 외부 DNS 역할 구분
2. Domain-Joined Client의 내부 DNS 사용
3. 불필요한 DNS Zone 수정 권한 제한
4. DNS 관리 권한 최소화
5. DNS Query 및 장애 상황 확인
6. DC01과 DC02의 DNS 이중화
7. Guest Network의 내부 DNS 접근 제한


## 18. DHCP Security Considerations

DHCP 환경에는 다음 원칙을 적용한다.

1. DHCP01에 고정 IP 사용
2. 승인된 DHCP Server만 운영
3. DHCP Scope별 IP 관리
4. 불필요한 DHCP Range 최소화
5. Lease 정보 확인
6. IP 충돌 발생 여부 확인
7. Guest Network과 내부 DHCP 환경 분리


## 19. Service Dependency

DHCP와 DNS는 다른 인프라 서비스와 다음과 같이 연결된다.

```text
DHCP
  |
  ↓
Client IP Assignment
  |
  ↓
DNS
  |
  ↓
Active Directory
  |
  +------→ File Server
  |
  +------→ Web Server
  |
  +------→ Database
  |
  +------→ Mail Server
  |
  +------→ Monitoring
```

특히 Active Directory는 DNS에 대한 의존성이 높기 때문에 DNS 장애가 Domain Authentication 및 여러 내부 서비스에 영향을 줄 수 있음을 고려한다.


## 20. Failure Scenario

### DC01 Failure

DC01에 장애가 발생한 경우:

```text
Client
  |
  ↓
DC02 DNS
  |
  ↓
Active Directory Authentication
```

DC02가 DNS 및 Domain Controller 역할을 수행할 수 있도록 구성한다.


### DHCP01 Failure

DHCP01에 장애가 발생하면 신규 Client의 IP 자동 할당에 문제가 발생할 수 있다.

기존 Lease를 보유한 Client는 일정 기간 동안 기존 IP를 사용할 수 있지만, 신규 IP 할당에는 영향을 받을 수 있다.

따라서 추후 프로젝트 확장 단계에서 DHCP 이중화 구성을 검토한다.


## 21. Validation Checklist

DHCP와 DNS 구축 후 다음 항목을 확인한다.

### DHCP

- DHCP01 정상 동작
- DHCP Scope 정상 생성
- Client IP 자동 할당
- Gateway 자동 할당
- DNS 정보 자동 할당
- Lease 정보 확인


### DNS

- DC01 DNS 정상 동작
- DC02 DNS 정상 동작
- Forward Lookup 정상 동작
- Reverse Lookup 정상 동작
- 서버 FQDN 조회 정상
- 외부 DNS 조회 정상


### Active Directory

- Client Domain Join 정상
- `suntech.local` 이름 해석 정상
- Domain Controller 검색 정상
- User Authentication 정상
- DC01 장애 시 DC02 사용 가능 여부 확인


### Network

- HQ User → DNS 통신
- Factory → DNS 통신
- IT Management → DNS 통신
- Guest → Internal DNS 차단
- Client → Internet 통신


## 22. Implementation Order

DHCP와 DNS는 다음 순서로 구축한다.

1. DC01 고정 IP 설정
2. DC02 고정 IP 설정
3. DC01 DNS 구성
4. Active Directory Domain 생성
5. DC02 Additional Domain Controller 구성
6. DC02 DNS 구성
7. DHCP01 VM 생성
8. DHCP01 고정 IP 설정
9. DHCP Server Role 설치
10. DHCP Scope 생성
11. DHCP Option 설정
12. DHCP Relay 구성
13. Client DHCP 테스트
14. DNS Forward Lookup 테스트
15. DNS Reverse Lookup 테스트
16. Domain Join 테스트
17. DC01 / DC02 장애 테스트


## 23. Final Design

SunTech의 DHCP와 DNS 환경은 다음과 같이 구성한다.

```text
                     Internet
                        |
                   pfSense
                        |
        +---------------+---------------+
        |               |               |
     HQ User          Factory       IT Management
  10.10.10.0/24     10.10.40.0/24    10.10.30.0/24
        |               |               |
        +---------------+---------------+
                        |
                  Server Network
                   10.10.20.0/24
                        |
        +---------------+----------------+
        |               |                |
      DC01            DC02            DHCP01
   10.10.20.10     10.10.20.11      10.10.20.12
      DNS             DNS             DHCP
        |
        +-------------------------------+
        |
   Active Directory
     suntech.local


Guest Network
10.10.50.0/24

→ Internet Access
→ Internal Network Access Denied
→ Internal AD DNS Access Denied
```

DHCP는 DHCP01에서 중앙 관리하고, DNS는 DC01과 DC02에서 제공한다.

Active Directory Client는 내부 DNS를 사용하며, DHCP를 통해 필요한 네트워크 설정을 자동으로 전달받는다.

이를 기반으로 사용자 인증, 파일 서버, 웹 서버, 데이터베이스, 메일 및 모니터링 환경을 단계적으로 연계한다.
