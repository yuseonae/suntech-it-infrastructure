# VMware VM Creation & Implementation

## 1. 구축 목적

SunTech 제조업체 IT 인프라 환경을 VMware Workstation Pro 기반의 가상화 환경으로 구축하였다.

실제 기업 환경의 서버 인프라를 가정하여 서버 네트워크와 사용자 네트워크를 분리하고, 각 서버에 역할별 VM을 구성할 수 있도록 기본 가상 네트워크 환경과 DC01 서버를 구축하였다.

### 구축 환경

| 항목 | 내용 |
|---|---|
| Hypervisor | VMware Workstation Pro 17 |
| Host CPU | Intel Core i7-9700F |
| Host RAM | 32GB |
| VM Storage | D:\VMware\SunTech |
| OS | Windows Server 2019 Standard Evaluation |
| Domain | suntech.local (예정) |
| Client Network | 10.10.10.0/24 |
| Server Network | 10.10.20.0/24 |

---

## 2. VMware 가상 네트워크 구성

SunTech IT 인프라의 사용자 영역과 서버 영역을 논리적으로 분리하기 위해 VMware의 Custom Network를 사용하였다.

### 2.1 Client Network

| 항목 | 설정 |
|---|---|
| VMware Network | VMnet10 |
| Network | 10.10.10.0/24 |
| Host Virtual Adapter | Disabled |
| VMware DHCP | Disabled |
| 용도 | 사용자 PC 및 테스트 클라이언트 |

VMnet10은 사용자 클라이언트 영역으로 사용한다.

향후 CLIENT01을 연결하여 DHCP01에서 IP를 할당받고, Active Directory 도메인 가입 및 GPO 적용을 테스트할 예정이다.

### 2.2 Server Network

| 항목 | 설정 |
|---|---|
| VMware Network | VMnet11 |
| Network | 10.10.20.0/24 |
| Host Virtual Adapter | Disabled |
| VMware DHCP | Disabled |
| 용도 | 서버 인프라 |

VMnet11은 Active Directory, DNS, DHCP, File Server, Web, Database, Mail, Monitoring 등의 서버가 위치하는 서버 영역으로 사용한다.

서버 IP는 DHCP가 아닌 정적 IP를 사용하여 주요 인프라 서버의 주소가 변경되지 않도록 구성한다.

---

## 3. VMware DHCP 비활성화

VMnet10과 VMnet11에서는 VMware가 제공하는 DHCP 기능을 사용하지 않는다.

실제 인프라 환경에서 DHCP 서비스를 별도의 DHCP01 서버가 담당하도록 설계했기 때문에 VMware DHCP가 동시에 동작할 경우 IP 주소가 중복 할당될 수 있다.

따라서 다음과 같이 구성하였다.

- VMnet10 → VMware DHCP Disabled
- VMnet11 → VMware DHCP Disabled
- 향후 DHCP01 → `10.10.20.12`
- Client Network DHCP → 향후 DHCP Relay를 통해 DHCP01에서 제공

이를 통해 가상화 환경의 네트워크 기능과 실제 서버 역할을 분리하였다.

---

## 4. DC01 VM 구축

### 4.1 VM 기본 사양

첫 번째 인프라 서버로 사용할 DC01 VM을 구축하였다.

DC01은 이후 Active Directory Domain Services와 DNS를 설치하여 `suntech.local` 도메인의 첫 번째 Domain Controller로 구성할 예정이다.

| 항목 | 설정 |
|---|---|
| VM Name | DC01 |
| OS | Windows Server 2019 Standard Evaluation |
| Installation | Desktop Experience |
| CPU | 2 vCPU |
| Memory | 2GB |
| Disk | 60GB |
| Network | VMnet11 |
| IP | 10.10.20.10 |
| DNS | 10.10.20.10 |
| Gateway | 미설정 |
| Storage Path | D:\VMware\SunTech\DC01 |

---

## 5. Windows Server 2019 설치

DC01 VM에 Windows Server 2019 Standard Evaluation (Desktop Experience)을 설치하였다.

설치 과정에서 60GB 가상 디스크를 운영체제 영역으로 사용하였다.

### 설치 후 기본 설정

- Windows Server 2019 설치
- Administrator 계정 비밀번호 설정
- 서버 호스트명 변경
- 네트워크 인터페이스 확인
- 정적 IP 설정
- DNS 설정
- 네트워크 연결 테스트

---

## 6. Hostname 변경

Windows Server 설치 직후 기본적으로 생성된 호스트명 대신 서버 역할을 명확하게 구분할 수 있도록 `DC01`로 변경하였다.

변경 후 명령 프롬프트에서 다음 명령으로 hostname을 확인하였다.

    hostname

확인 결과:

    DC01

서버 역할에 따른 명명 규칙을 적용하여 이후 DC02, DHCP01, FILE01 등의 서버를 동일한 방식으로 구성할 예정이다.

---

## 7. DC01 네트워크 설정

DC01은 Server Network인 `10.10.20.0/24`에 연결하였다.

### 최종 네트워크 설정

| 항목 | 값 |
|---|---|
| IP Address | 10.10.20.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 미설정 |
| Preferred DNS | 10.10.20.10 |
| DHCP | Disabled |

DC01은 현재 Firewall/Router가 구축되지 않은 초기 구성 단계이므로 Default Gateway를 설정하지 않았다.

향후 Firewall/Router 구축 후 네트워크 정책에 따라 필요한 서버에 Gateway를 적용할 예정이다.

---

## 8. 네트워크 설정 검증

네트워크 설정이 정상적으로 적용되었는지 명령 프롬프트를 통해 확인하였다.

### 8.1 IP 설정 확인

    ipconfig /all

확인 항목:

- IP Address: `10.10.20.10`
- Subnet Mask: `255.255.255.0`
- DHCP: Disabled

DNS 설정은 다음 명령으로 추가 확인하였다.

    netsh interface ipv4 show dnsservers

확인 결과 `10.10.20.10`이 정적 DNS 서버로 설정되어 있음을 확인하였다.

### 8.2 자기 자신에 대한 Ping 테스트

    ping 10.10.20.10

테스트 결과:

- 전송: 4
- 수신: 4
- 손실: 0%
- Packet Loss: 0%

DC01의 TCP/IP 기본 설정이 정상적으로 적용되었음을 확인하였다.

---

## 9. 구축 중 발생한 문제 및 해결

### 문제: DC01 VM 최초 부팅 시 Windows 설치 화면이 표시되지 않음

#### Symptom

DC01 VM을 처음 실행했을 때 Windows Server 설치 화면 대신 다음과 같은 메시지가 표시되었다.

    EFI Network...
    Time out

VM이 설치 미디어가 아닌 네트워크 부팅을 시도하는 상황으로 판단하였다.

#### 원인

VM의 부팅 과정에서 Windows Server 설치 ISO가 정상적으로 부팅 미디어로 사용되지 않았다.

#### 조치

VMware에서 `Power On to Firmware`를 사용하여 VM의 UEFI Firmware 화면으로 진입한 후 부팅 장치를 확인하였다.

이후 Windows Server 2019 설치 ISO를 정상적으로 연결하고 다시 부팅하여 Windows Server 설치 화면이 표시되는 것을 확인하였다.

#### 결과

Windows Server 2019 Standard Evaluation (Desktop Experience) 설치를 정상적으로 완료하였다.

### 정리

이번 문제를 통해 VM 생성 단계에서 다음 항목을 사전에 확인할 필요가 있음을 확인하였다.

- 설치 ISO 연결 여부
- CD/DVD 장치 연결 상태
- Boot Order
- UEFI Firmware 설정
- VM 전원 상태

단순히 VM 생성만 하는 것이 아니라 실제 부팅 과정에서 발생하는 문제의 원인을 확인하고 해결하는 과정을 기록하였다.

---

## 10. 구축 결과

현재까지 다음과 같은 VMware 및 DC01 기본 구축을 완료하였다.

### VMware

- [x] SunTech 전용 VM 저장 경로 생성
- [x] Client Network VMnet10 구성
- [x] Server Network VMnet11 구성
- [x] VMware DHCP 비활성화
- [x] 네트워크 영역 분리

### DC01

- [x] DC01 VM 생성
- [x] Windows Server 2019 설치
- [x] 서버 hostname 변경
- [x] Static IP 설정
- [x] DNS 설정
- [x] IP 설정 검증
- [x] Ping 테스트

### 다음 구축 단계

DC01의 기본 환경 구축을 완료한 후 다음 단계로 Active Directory Domain Services와 DNS 역할을 설치한다.

이후 `suntech.local` 도메인을 생성하고 DC01을 첫 번째 Domain Controller로 구성한다.

---

## 11. Evidence

실제 구축 과정에서 주요 설정 및 검증 결과를 캡처하여 별도로 관리하였다.

원본 캡처는 로컬 환경에서 관리하고, GitHub에는 포트폴리오에서 구축 과정을 확인하는 데 필요한 핵심 화면을 선별하여 업로드한다.

### 관련 캡처

- VMware 네트워크 구성
- DC01 VM 생성 및 하드웨어 설정
- Windows Server 설치
- DC01 hostname 변경
- DC01 IP 설정
- IP 설정 검증
- DNS 설정 검증
- Ping 테스트

상세 캡처는 `docs/screenshots/vmware/` 및 `docs/screenshots/dc/`에 정리한다.
