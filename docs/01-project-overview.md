# 01. Project Overview

## 1. Project Overview

본 프로젝트는 약 200명 규모의 제조기업 **SunTech**의 IT 인프라 환경을 가정하여
기업 네트워크, 서버, Active Directory, 보안, 파일 서비스, 백업 및
시스템 운영 환경을 단계적으로 설계하고 구현하는 프로젝트이다.

단순한 서버 구축에 그치지 않고,
실제 제조기업에서 발생할 수 있는 IT 인프라 운영 요구사항을 정의한 후
요구사항에 적합한 인프라를 설계하고 VMware 환경에서 주요 시스템을 직접 구축한다.

---

## 2. Company Information

| Category | Description |
|---|---|
| Company | SunTech |
| Industry | Manufacturing |
| Employees | Approximately 200 |
| Sites | Headquarters + 1 Factory |
| IT Environment | Windows-based enterprise IT infrastructure |

---

## 3. Organization

SunTech는 다음과 같은 6개 부서로 구성되어 있다.

| Department | Employees | Main Role |
|---|---:|---|
| Management | 25 | HR, General Affairs, Finance |
| Sales | 25 | Customer and Sales Management |
| Production | 75 | Production Planning and Manufacturing |
| Quality | 25 | Quality Control and Inspection |
| R&D | 40 | Product Development and Engineering |
| IT | 10 | Infrastructure, System and Security Management |
| **Total** | **200** | |

---

## 4. Organizational Structure

```text
SunTech
│
├── Headquarters
│   ├── Management
│   ├── Sales
│   └── IT
│
└── Factory
    ├── Production
    ├── Quality
    └── R&D

---

## 5. Project Scope

본 프로젝트에서는 제조기업의 IT 인프라를 다음과 같은 영역으로 나누어 단계적으로 설계하고 구축한다.

### 5.1 Network

- IP Address Design
- VLAN Design
- Headquarters / Factory Network
- Server Network
- User Network
- IT Management Network

### 5.2 Server Infrastructure

- Windows Server
- Active Directory Domain Services
- DNS
- DHCP
- File Server
- FSRM
- Backup Server

### 5.3 Active Directory

- Domain Design
- OU Structure
- User and Group Management
- AGDLP
- Group Policy

### 5.4 Security

- Password Policy
- Account Lockout Policy
- USB Control
- File Access Control
- Audit Policy
- Least Privilege

### 5.5 Backup & Recovery

- Backup Policy
- Backup Schedule
- File Recovery
- Server Recovery

### 5.6 System Operations

- User Account Provisioning
- Employee Offboarding
- Permission Management
- Troubleshooting
- Incident Response

---

## 6. Project Objectives

본 프로젝트의 목표는 다음과 같다.

1. 제조기업의 IT 인프라 요구사항을 정의한다.
2. 기업 환경에 적합한 네트워크 및 서버 구조를 설계한다.
3. Windows Server 기반 인프라를 구축한다.
4. Active Directory를 활용하여 사용자와 조직을 중앙 관리한다.
5. 부서별 파일 접근 권한 및 보안 정책을 구현한다.
6. 데이터 보호를 위한 백업 및 복구 환경을 구축한다.
7. 실제 IT 운영 상황을 가정한 장애 대응 시나리오를 구현한다.
8. 구축 결과를 문서화하여 기업 IT 인프라 설계 및 운영 역량을 검증한다.
