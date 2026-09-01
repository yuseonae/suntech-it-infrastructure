# 05. File Server Design

## 1. Purpose

SunTech의 부서별 업무 자료를 중앙에서 관리하고 사용자별 접근 권한을 통제하기 위해 File Server를 구축한다.

File Server는 Active Directory의 Security Group과 연계하여 부서별 파일 접근 권한을 관리한다.

단순한 파일 공유 환경이 아니라 다음 요소를 함께 적용한다.

- SMB File Sharing
- NTFS Permission
- Active Directory Security Group
- AGDLP Permission Model
- FSRM
- File Access Auditing
- Backup


## 2. File Server Architecture

File Server는 별도의 서버인 FILE01에서 운영한다.

### FILE01

Hostname:

`FILE01`

IP Address:

`10.10.20.20`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

Primary DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

Domain:

`suntech.local`

FILE01은 Active Directory Domain에 Join하여 사용자 및 Security Group 정보를 이용한다.


## 3. File Server Role

FILE01의 주요 역할은 다음과 같다.

- File Server
- SMB File Sharing
- Department Shared Folders
- Personal Folder
- FSRM
- File Access Auditing

파일 서버는 사용자 PC에 자료를 직접 저장하는 대신 중앙 저장소 역할을 수행한다.


## 4. Department Structure

부서별 업무 자료를 분리하여 관리한다.

대상 부서:

- Management
- IT
- Production
- Quality
- RnD
- Sales
- HR
- Finance


## 5. Folder Structure

FILE01의 기본 폴더 구조는 다음과 같이 구성한다.

    D:\
    └── Shares
        ├── Departments
        │   ├── Management
        │   ├── IT
        │   ├── Production
        │   ├── Quality
        │   ├── RnD
        │   ├── Sales
        │   ├── HR
        │   └── Finance
        │
        ├── Public
        │
        └── Personal

각 부서의 업무 자료는 해당 부서 폴더에서 관리한다.


## 6. Department Folder

부서별 폴더는 다음과 같이 구성한다.

    D:\Shares\Departments\Management
    D:\Shares\Departments\IT
    D:\Shares\Departments\Production
    D:\Shares\Departments\Quality
    D:\Shares\Departments\RnD
    D:\Shares\Departments\Sales
    D:\Shares\Departments\HR
    D:\Shares\Departments\Finance

각 폴더에는 해당 부서의 업무 자료를 저장한다.


## 7. Public Folder

공통 자료를 위한 Public 폴더를 별도로 구성한다.

경로:

`D:\Shares\Public`

Public 폴더는 모든 일반 사용자가 읽을 수 있도록 구성할 수 있으며, 실제 쓰기 권한은 별도의 Security Group을 통해 관리한다.

예상 그룹:

- GG-All-Users
- DL-Public-Share-RW

실제 권한은 업무 목적에 따라 Read 또는 Modify로 구분한다.


## 8. Personal Folder

필요한 경우 사용자 개인 자료를 저장할 수 있도록 Personal 영역을 구성한다.

경로:

`D:\Shares\Personal`

사용자별 하위 폴더를 생성한다.

예:

    D:\Shares\Personal\SUN0001
    D:\Shares\Personal\SUN0002
    D:\Shares\Personal\SUN0003

개인 폴더는 해당 사용자에게만 접근 권한을 부여한다.


## 9. SMB Share Design

부서별 폴더는 SMB를 이용하여 네트워크 공유한다.

예:

    \\FILE01\Production
    \\FILE01\Quality
    \\FILE01\RnD
    \\FILE01\Sales
    \\FILE01\HR
    \\FILE01\Finance

공통 폴더:

    \\FILE01\Public

개인 폴더:

    \\FILE01\Personal

사용자는 IP 주소 대신 서버 이름을 이용하여 파일 서버에 접근한다.


## 10. Share Naming Convention

공유 이름은 실제 경로와 사용 목적을 쉽게 파악할 수 있도록 구성한다.

| Share Name | Path | Purpose |
|---|---|---|
| Management | D:\Shares\Departments\Management | Management Files |
| IT | D:\Shares\Departments\IT | IT Files |
| Production | D:\Shares\Departments\Production | Production Files |
| Quality | D:\Shares\Departments\Quality | Quality Files |
| RnD | D:\Shares\Departments\RnD | R&D Files |
| Sales | D:\Shares\Departments\Sales | Sales Files |
| HR | D:\Shares\Departments\HR | HR Files |
| Finance | D:\Shares\Departments\Finance | Finance Files |
| Public | D:\Shares\Public | Common Files |
| Personal | D:\Shares\Personal | Personal Files |


## 11. Active Directory Group Integration

File Server 권한은 Active Directory Security Group과 연계한다.

사용자에게 직접 NTFS 권한을 부여하는 방식은 지양한다.

기본적인 구조:

    User Account
          ↓
    Global Group
          ↓
    Domain Local Group
          ↓
    Share / NTFS Permission

이를 통해 사용자의 부서 이동이나 권한 변경이 발생했을 때 Group Membership만 변경하여 관리할 수 있도록 한다.


## 12. AGDLP Model

파일 서버 권한에는 AGDLP 모델을 적용한다.

### AGDLP

    Account
       ↓
    Global Group
       ↓
    Domain Local Group
       ↓
    Permission

예:

    SUN0001
       ↓
    GG-Production-Users
       ↓
    DL-Production-Share-RW
       ↓
    Production Folder


### Example Groups

Global Group:

- GG-Production-Users
- GG-Quality-Users
- GG-RnD-Users
- GG-Sales-Users
- GG-HR-Users
- GG-Finance-Users

Domain Local Group:

- DL-Production-Share-RW
- DL-Quality-Share-RW
- DL-RnD-Share-RW
- DL-Sales-Share-RW
- DL-HR-Share-RW
- DL-Finance-Share-RW

## 13. Department Permission Matrix

기본적인 부서별 권한은 다음과 같이 구성한다.

| Department | Own Folder | Other Department Folder | Public |
|---|---|---|---|
| Management | RW | RO / 필요 시 | RW |
| IT | RW | 관리 목적에 따라 | RW |
| Production | RW | Deny | RW |
| Quality | RW | Deny | RW |
| RnD | RW | Deny | RW |
| Sales | RW | Deny | RW |
| HR | RW | Deny | RW |
| Finance | RW | Deny | RW |

기본적으로 일반 사용자는 자신의 부서 폴더에만 접근할 수 있도록 구성한다.

관리 및 업무상 필요한 예외 권한은 별도의 Security Group으로 관리한다.


## 14. NTFS Permission Design

NTFS 권한은 Security Group을 대상으로 부여한다.

기본적으로 다음 원칙을 적용한다.

### Department Folder

해당 부서의 Domain Local Group:

`Modify`

IT 관리 그룹:

`Full Control`

Administrators:

`Full Control`

기타 일반 사용자:

`No Access`

예:

    D:\Shares\Departments\Production

    DL-Production-Share-RW
        → Modify

    IT Admin Group
        → Full Control

    Administrators
        → Full Control


## 15. Share Permission Design

Share Permission과 NTFS Permission을 함께 고려한다.

기본적인 설계는 Share Permission을 넓게 설정하고 실제 세부 권한은 NTFS Permission으로 관리하는 방식으로 구성한다.

예:

    Share Permission
    Authenticated Users
        → Full Control

    NTFS Permission
    Production Users
        → Modify

    IT Admins
        → Full Control

최종적인 접근 권한은 Share Permission과 NTFS Permission의 조합으로 결정된다.


## 16. Permission Management Principle

사용자에게 직접 권한을 부여하는 방식은 최소화한다.

잘못된 방식:

    SUN0001
       ↓
    Production Folder
       ↓
    Modify

권장 방식:

    SUN0001
       ↓
    GG-Production-Users
       ↓
    DL-Production-Share-RW
       ↓
    Production Folder
       ↓
    Modify

이 구조를 통해 권한 관리의 일관성과 확장성을 확보한다.


## 17. Access-Based Enumeration

사용자가 권한이 없는 공유 폴더를 불필요하게 확인하지 않도록 Access-Based Enumeration을 검토한다.

예:

Production 사용자는 필요한 공유 폴더만 확인할 수 있도록 구성한다.

    Production
    Public

이를 통해 사용자에게 불필요한 폴더 정보를 노출하는 것을 줄인다.


## 18. FSRM

File Server Resource Manager(FSRM)를 이용하여 파일 서버의 저장 공간과 파일 유형을 관리한다.

주요 기능:

- Quota Management
- File Screening
- Storage Reports


## 19. Quota Design

부서별 저장 공간 사용량을 관리하기 위해 Quota를 적용한다.

초기에는 테스트 목적의 제한을 설정하고 실제 구축 과정에서 적절한 값을 검토한다.

| Folder | Quota |
|---|---:|
| Production | 20 GB |
| Quality | 15 GB |
| RnD | 30 GB |
| Sales | 15 GB |
| HR | 10 GB |
| Finance | 10 GB |
| IT | 20 GB |

실제 용량은 VM 환경과 호스트 저장 공간을 고려하여 조정한다.


## 20. File Screening

업무와 관련 없는 파일 또는 불필요하게 큰 파일의 저장을 제한하기 위해 File Screening을 적용한다.

예를 들어 다음 파일 유형을 제한 대상으로 검토한다.

- 실행 파일
- 악성 스크립트 파일
- 불필요한 설치 파일
- 비정상적으로 큰 파일

단, 업무상 필요한 파일을 차단하지 않도록 실제 운영 환경을 가정하여 예외 정책을 함께 고려한다.


## 21. Storage Reports

FSRM Storage Reports를 이용하여 다음 정보를 확인한다.

- 폴더별 저장 공간 사용량
- 파일 유형별 사용량
- 대용량 파일
- 최근 접근 파일
- 파일 생성 및 수정 현황

이를 통해 저장 공간을 지속적으로 관리할 수 있도록 한다.


## 22. File Access Auditing

중요한 부서 파일에 대한 접근 기록을 확인할 수 있도록 감사 정책을 구성한다.

감사 대상:

- 파일 접근
- 파일 수정
- 파일 삭제
- 권한 변경

예:

    User: SUN0001
    Resource: \\FILE01\Production\production-plan.xlsx
    Action: Modify
    Result: Success

필요한 감사 이벤트는 Windows Security Log에 기록하도록 구성한다.


## 23. Backup Integration

FILE01의 데이터를 백업하기 위해 BACKUP01과 연계한다.

Backup Server:

`BACKUP01`

IP:

`10.10.20.30`

기본적인 구조:

    FILE01
      |
      | Backup
      ↓
    BACKUP01

백업 대상:

- Department Shares
- Public
- Personal
- File Server Configuration


## 24. Backup Strategy

초기 프로젝트에서는 다음 백업 전략을 적용한다.

### Full Backup

주기적으로 전체 데이터를 백업한다.

### Incremental / Differential Backup

변경된 데이터를 중심으로 추가 백업하는 방식을 검토한다.

실제 구현에서는 Windows Server Backup을 활용하여 기본적인 백업 및 복구 시나리오를 구축한다.

## 25. File Server Failure Scenario

FILE01에 장애가 발생하는 경우 파일 공유 서비스에 영향을 받을 수 있다.

따라서 BACKUP01에 백업 데이터를 보관하여 장애 발생 시 복구할 수 있도록 한다.

기본적인 복구 흐름:

    FILE01 Failure
         ↓
    Identify Failure
         ↓
    Prepare Recovery Environment
         ↓
    Restore Backup
         ↓
    Verify NTFS / Share Permission
         ↓
    Verify User Access


## 26. Permission Testing

권한 설정 후 반드시 부서별 접근 테스트를 수행한다.

예:

### Production User

    \\FILE01\Production
    → Access Granted

    \\FILE01\Finance
    → Access Denied


### Finance User

    \\FILE01\Finance
    → Access Granted

    \\FILE01\Production
    → Access Denied


### IT Administrator

    \\FILE01\Production
    → Access Granted

    \\FILE01\Finance
    → Access Granted

관리자는 업무 수행에 필요한 범위에서 부서별 파일 서버를 관리할 수 있도록 별도의 관리 권한을 부여한다.


## 27. Permission Troubleshooting

권한 문제가 발생하면 다음 순서로 확인한다.

1. User Account 확인
2. Global Group Membership 확인
3. Domain Local Group Membership 확인
4. Share Permission 확인
5. NTFS Permission 확인
6. Inheritance 확인
7. Explicit Deny 확인
8. Effective Access 확인
9. User Logoff / Logon
10. Access Test

권한 변경 후에는 사용자의 로그온 세션에 이전 권한 정보가 남아 있을 수 있으므로 재로그인 또는 필요한 경우 관련 세션을 갱신한 후 테스트한다.


## 28. Security Principles

File Server에는 다음 보안 원칙을 적용한다.

1. 최소 권한 원칙
2. 사용자 직접 권한 부여 최소화
3. AD Security Group 기반 권한 관리
4. AGDLP 적용
5. 부서별 폴더 분리
6. 관리자 계정 분리
7. 파일 접근 감사
8. FSRM을 통한 저장 공간 관리
9. 중요 데이터 백업
10. 정기적인 권한 검토


## 29. Implementation Scope

이번 프로젝트에서는 다음 항목을 실제로 구축한다.

- FILE01
- Windows Server File Server Role
- SMB Share
- Department Folder
- Public Folder
- Active Directory Security Group 연계
- AGDLP
- Share Permission
- NTFS Permission
- Access-Based Enumeration
- FSRM Quota
- File Screening
- Storage Report
- File Access Auditing
- BACKUP01 연계
- File Restore Test


## 30. Implementation Order

File Server 구축은 다음 순서로 진행한다.

1. FILE01 VM 생성
2. Windows Server 설치
3. Static IP 설정
4. DNS 설정
5. `suntech.local` Domain Join
6. File Server Role 설치
7. Data Disk 구성
8. Folder Structure 생성
9. SMB Share 생성
10. Active Directory Security Group 생성
11. Global Group에 사용자 추가
12. Domain Local Group 생성
13. AGDLP 구조 구성
14. Share Permission 설정
15. NTFS Permission 설정
16. Permission Inheritance 확인
17. Access-Based Enumeration 설정
18. FSRM 설치
19. Quota 설정
20. File Screening 설정
21. Storage Report 설정
22. File Access Auditing 구성
23. 부서별 접근 테스트
24. 권한 오류 테스트
25. BACKUP01 백업 구성
26. File Restore Test


## 31. Validation Checklist

### Server

- FILE01 정상 동작
- Domain Join 정상
- DNS Resolution 정상
- File Server Role 정상


### SMB

- Share 정상 생성
- Client에서 Share 접근 가능
- 부서별 Share 접근 확인


### Permission

- Production → Production 접근 가능
- Production → Finance 접근 차단
- Finance → Finance 접근 가능
- Finance → Production 접근 차단
- IT Admin → 필요한 부서 폴더 접근 가능


### AGDLP

- User → Global Group
- Global Group → Domain Local Group
- Domain Local Group → Resource Permission


### FSRM

- Quota 정상 동작
- File Screening 정상 동작
- Storage Report 생성


### Audit

- 파일 접근 이벤트 기록
- 파일 수정 이벤트 기록
- 파일 삭제 이벤트 기록
- 권한 변경 이벤트 기록


### Backup

- FILE01 Backup 정상
- Backup Data 확인
- File Restore 정상
- 복구 후 Permission 정상


## 32. Final Design

SunTech File Server는 Active Directory 기반 권한 관리와 중앙 파일 저장소를 결합한 구조로 구성한다.

    Active Directory
        suntech.local
               |
        +------+------+
        |             |
    Global Groups   Domain Local Groups
        |             |
        +------+------+
               |
               ↓
             FILE01
          10.10.20.20
               |
      +--------+--------+
      |        |        |
    Department Public Personal
      Shares    Share    Share
        |
    +---+---+---+---+---+
    |   |   |   |   |   |
   PROD QA  RnD Sales HR Finance

FILE01은 AD Security Group과 AGDLP 모델을 기반으로 부서별 파일 접근 권한을 관리한다.

FSRM을 이용하여 저장 공간과 파일 유형을 관리하고, Windows Security Audit을 이용하여 중요 파일의 접근 및 변경 이력을 기록한다.

또한 BACKUP01과 연계하여 파일 서버 데이터를 백업하고 장애 발생 시 복구할 수 있는 환경을 구축한다.

이를 통해 단순한 파일 공유 서버가 아니라 사용자 인증, 권한 관리, 보안, 감사 및 백업이 연결된 기업형 File Server 환경을 구현한다.
