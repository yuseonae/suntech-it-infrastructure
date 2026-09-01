# 02. IP Address Plan

## 1. IP Addressing Overview

SunTech의 내부 네트워크는 사설 IP 주소 대역 `10.10.0.0/16`을 사용한다.

네트워크 영역별로 `/24` 대역을 할당하여 IP 주소 관리와 확장성을 확보한다.


## 2. VLAN IP Address Plan

| VLAN ID | Network | Subnet Mask | Gateway | Purpose |
|---:|---|---|---|---|
| 10 | `10.10.10.0/24` | `255.255.255.0` | `10.10.10.1` | HQ User |
| 20 | `10.10.20.0/24` | `255.255.255.0` | `10.10.20.1` | Server |
| 30 | `10.10.30.0/24` | `255.255.255.0` | `10.10.30.1` | IT Management |
| 40 | `10.10.40.0/24` | `255.255.255.0` | `10.10.40.1` | Factory |
| 50 | `10.10.50.0/24` | `255.255.255.0` | `10.10.50.1` | Guest |


## 3. IP Address Allocation Rules

각 네트워크의 IP 주소는 역할에 따라 구분하여 관리한다.

### `.1`

각 VLAN의 기본 Gateway로 사용한다.

### `.2 ~ .49`

네트워크 장비 및 주요 인프라 장비를 위한 영역으로 사용한다.

### `.50 ~ .99`

향후 추가되는 고정 IP 장비 및 서버를 위한 예비 영역으로 사용한다.

### `.100 ~ .199`

사용자 및 일반 클라이언트의 DHCP 할당 영역으로 사용한다.

### `.200 ~ .254`

향후 확장을 위한 예비 영역으로 유지한다.


## 4. Server IP Address Plan

현재까지 정의된 주요 인프라 서버는 다음과 같이 고정 IP를 사용한다.

| Hostname | IP Address | Role |
|---|---|---|
| DC01 | `10.10.20.10` | Active Directory / DNS / DHCP |
| DC02 | `10.10.20.11` | Active Directory / DNS |
| FILE01 | `10.10.20.20` | File Server / FSRM |
| BACKUP01 | `10.10.20.30` | Backup Server |

추가 서버가 구축될 경우 `10.10.20.40 ~ 10.10.20.99` 영역을 우선 활용한다.

> 참고: 메일 서버, 모니터링 서버, 로그 서버, DB 서버 및 웹 서버 등은 향후 서버 및 서비스 설계 단계에서 최종 검토 후 추가한다.


## 5. DHCP Address Plan

### 5.1 HQ User

```text
Network : 10.10.10.0/24
Gateway : 10.10.10.1
DHCP    : 10.10.10.100 - 10.10.10.199
DNS     : 10.10.20.10
```


### 5.2 Factory

```text
Network : 10.10.40.0/24
Gateway : 10.10.40.1
DHCP    : 10.10.40.100 - 10.10.40.199
DNS     : 10.10.20.10
```


### 5.3 IT Management

```text
Network : 10.10.30.0/24
Gateway : 10.10.30.1
DHCP    : 10.10.30.100 - 10.10.30.149
DNS     : 10.10.20.10
```


### 5.4 Guest

```text
Network : 10.10.50.0/24
Gateway : 10.10.50.1
DHCP    : 10.10.50.100 - 10.10.50.199
```

Guest 네트워크는 내부 AD DNS를 직접 사용하지 않고 외부 DNS 또는 방화벽에서 제공하는 DNS 서비스를 사용하는 것을 원칙으로 한다.


## 6. DNS Configuration

내부 Windows 도메인 환경에서는 Active Directory와 연동된 내부 DNS 서버를 사용한다.

Primary DNS:

```text
10.10.20.10
```

Secondary DNS:

```text
10.10.20.11
```

사용자 및 서버 네트워크에서는 내부 DNS를 사용하여 Active Directory 도메인 이름을 정상적으로 해석할 수 있도록 구성한다.


## 7. Address Allocation Examples

### HQ User PC

```text
IP Address : 10.10.10.100
Subnet     : 255.255.255.0
Gateway    : 10.10.10.1
DNS        : 10.10.20.10
```


### Factory PC

```text
IP Address : 10.10.40.100
Subnet     : 255.255.255.0
Gateway    : 10.10.40.1
DNS        : 10.10.20.10
```


### IT Administrator PC

```text
IP Address : 10.10.30.100
Subnet     : 255.255.255.0
Gateway    : 10.10.30.1
DNS        : 10.10.20.10
```


## 8. IP Address Management Principles

IP 주소 관리 시 다음 원칙을 적용한다.

1. 서버 및 주요 인프라 장비는 고정 IP를 사용한다.
2. 사용자 PC는 DHCP를 통해 자동으로 IP를 할당받는다.
3. 각 VLAN의 Gateway 주소는 `.1`로 통일한다.
4. DHCP 주소와 고정 IP 주소가 중복되지 않도록 관리한다.
5. IP 주소 할당 현황을 문서화하여 관리한다.
6. 향후 서버 및 네트워크 확장을 고려하여 일부 IP 주소를 예비 영역으로 유지한다.
7. 네트워크 변경 시 IP Address Plan 문서를 함께 갱신한다.
