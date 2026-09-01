# 07. Mail Server Design

## 1. Purpose

SunTech의 사내 업무 환경에서 사용할 수 있는 메일 서비스를 구축하여 사용자 간 업무 메일 송수신 환경을 구성한다.

메일 서버를 통해 기업 IT 인프라에서 사용되는 SMTP, IMAP, DNS, 사용자 계정, 인증, 로그 및 보안 정책을 직접 구성하고 운영하는 것을 목표로 한다.

이번 프로젝트에서는 대규모 상용 메일 시스템을 구축하기보다는 실제 실습 가능한 규모의 내부 메일 환경을 구성한다.


## 2. Mail Server Architecture

메일 서버는 별도의 Linux Server인 MAIL01에서 운영한다.

### MAIL01

Hostname:

`MAIL01`

IP Address:

`10.10.20.80`

Subnet Mask:

`255.255.255.0`

Gateway:

`10.10.20.1`

Primary DNS:

`10.10.20.10`

Secondary DNS:

`10.10.20.11`

Mail Domain:

`suntech.local`


## 3. Mail Server Software

MAIL01은 Linux 기반으로 구성한다.

주요 소프트웨어:

- Postfix
- Dovecot

Postfix는 SMTP 기반 메일 송수신을 담당하고, Dovecot은 IMAP 기반 메일 조회 및 사용자 메일함 접근을 담당한다.


## 4. Mail Service Role

MAIL01의 주요 역할은 다음과 같다.

- SMTP Server
- IMAP Server
- Internal Mail Delivery
- User Mailbox Management
- Mail Log Management
- Mail Authentication


## 5. Mail Service Flow

기본적인 메일 흐름은 다음과 같다.

    Sender
       |
       | SMTP
       ↓
    MAIL01
       |
       | Mail Delivery
       ↓
    Recipient Mailbox
       |
       | IMAP
       ↓
    Mail Client

사용자는 메일 클라이언트를 통해 MAIL01에 접속하여 메일을 송수신한다.


## 6. SMTP

SMTP는 메일을 전송하는 데 사용한다.

기본적으로 다음 포트를 사용한다.

| Protocol | Port | Purpose |
|---|---:|---|
| SMTP | 25 | Server-to-Server Mail Transfer |
| Submission | 587 | Authenticated Mail Submission |
| SMTPS | 465 | Secure Mail Submission 검토 |

이번 프로젝트에서는 사용자 메일 발송에 `587` 포트를 우선 사용한다.


## 7. IMAP

Dovecot은 IMAP을 이용하여 사용자의 메일함을 제공한다.

주요 포트:

| Protocol | Port | Purpose |
|---|---:|---|
| IMAP | 143 | Mailbox Access |
| IMAPS | 993 | Secure Mailbox Access |

실제 사용자 메일 클라이언트에서는 보안을 고려하여 `993` 포트를 우선 사용한다.


## 8. Mail Domain

메일 주소는 다음 형식을 사용한다.

`user@suntech.local`

예:

`sun0001@suntech.local`

사용자의 메일 주소와 Active Directory 계정의 관계를 고려하여 사용자 식별 체계를 일관되게 유지한다.


## 9. User Mailbox

초기 테스트 사용자는 다음과 같이 구성한다.

| Employee No | Mail Address | Department |
|---|---|---|
| SUN0001 | sun0001@suntech.local | Production |
| SUN0002 | sun0002@suntech.local | Quality |
| SUN0003 | sun0003@suntech.local | IT |
| SUN0004 | sun0004@suntech.local | Sales |

실제 구축 단계에서 테스트 계정을 생성하여 메일 송수신을 검증한다.


## 10. Authentication

메일 서비스 사용자는 인증을 거친 후 메일을 송수신하도록 구성한다.

기본 구조:

    User
      |
      ↓
    Mail Client
      |
      ↓
    MAIL01
      |
      ↓
    Authentication
      |
      ↓
    Mail Service

SMTP Submission에서는 인증되지 않은 사용자의 메일 발송을 허용하지 않는다.

## 11. Active Directory Integration

메일 계정 인증은 향후 Active Directory와 연계하는 방안을 검토한다.

기본적인 구조:

    Active Directory
          |
          ↓
      User Account
          |
          ↓
        MAIL01
          |
          ↓
      Mail Service

초기 구축에서는 Linux 로컬 계정을 이용하여 메일 서비스를 먼저 검증할 수 있다.

메일 서비스가 정상적으로 동작한 후 LDAP 또는 Active Directory 연계를 추가하는 방식으로 난이도를 단계적으로 높인다.


## 12. DNS Configuration

메일 서비스는 DNS와 밀접하게 연결되므로 내부 DNS에 MAIL01 정보를 등록한다.

A Record:

`mail.suntech.local`

→ `10.10.20.80`

또는:

`mail01.suntech.local`

→ `10.10.20.80`

사용자는 IP 주소가 아닌 Mail Server의 Hostname을 이용한다.


## 13. Mail Client Configuration

테스트용 PC에서는 다음과 같이 메일 클라이언트를 설정한다.

### Incoming Mail

Protocol:

`IMAPS`

Server:

`mail.suntech.local`

Port:

`993`

### Outgoing Mail

Protocol:

`SMTP Submission`

Server:

`mail.suntech.local`

Port:

`587`

SMTP 인증:

`Required`


## 14. Internal Mail Flow

SunTech 내부 사용자 간 메일은 MAIL01을 통해 전달한다.

예:

    SUN0001
    sun0001@suntech.local
          |
          ↓
        MAIL01
          |
          ↓
    sun0002@suntech.local
    SUN0002

인터넷을 거치지 않고 내부 Mail Server에서 직접 처리한다.


## 15. External Mail Flow

외부 메일 환경은 실제 인터넷 메일 서버와 직접 연동하기보다 구조를 이해하고 테스트할 수 있도록 설계한다.

기본적인 외부 메일 흐름:

    Internal User
         |
         ↓
       MAIL01
         |
         ↓
    External Mail Server
         |
         ↓
    External User

외부 메일을 실제로 운영하기 위해서는 공인 IP, DNS MX Record, SPF, DKIM, DMARC, TLS 및 Reverse DNS 등의 추가 구성이 필요하다.

이번 프로젝트에서는 외부 메일 송수신은 선택적인 확장 범위로 두고 내부 메일 환경을 우선 구축한다.


## 16. Mail Relay Security

메일 서버에서 가장 중요한 보안 항목 중 하나는 Open Relay 방지이다.

인증되지 않은 외부 사용자가 MAIL01을 이용하여 임의의 외부 주소로 메일을 발송하지 못하도록 제한한다.

기본 원칙:

- 인증된 사용자만 Submission 사용
- 외부 사용자의 임의 Relay 차단
- 내부 Network 정책 설정
- SMTP Relay 대상 제한


## 17. SMTP Relay Policy

기본 정책:

    Internal Authenticated User
             ↓
        MAIL01
             ↓
        Mail Delivery

외부의 인증되지 않은 사용자가:

    External User
          ↓
       MAIL01
          ↓
    External Recipient

형태로 메일을 중계하는 것은 차단한다.


## 18. Spam Protection

초기 프로젝트에서는 복잡한 Anti-Spam 시스템을 별도로 구축하지 않는다.

대신 기본적인 보호 정책을 적용한다.

- SMTP Relay 제한
- 인증 사용자 기반 발송
- 외부 연결 제한
- 발신자 정보 검증
- Mail Log 확인

추후 필요하면 SpamAssassin 등의 Anti-Spam 솔루션을 추가할 수 있다.


## 19. TLS

메일 계정 및 메일 내용 보호를 위해 TLS 적용을 검토한다.

주요 대상:

- SMTP Submission
- IMAP

예상 구조:

    Mail Client
        |
        | TLS
        ↓
      MAIL01

실제 구축 단계에서는 테스트용 인증서 또는 내부 CA 인증서를 이용하여 TLS 환경을 구성한다.


## 20. Mailbox Storage

사용자 메일함은 MAIL01의 별도 데이터 영역에 저장한다.

예상 구조:

    /var/mail/
        ├── sun0001
        ├── sun0002
        ├── sun0003
        └── sun0004

실제 저장 경로는 Dovecot의 Mailbox 설정에 따라 구성한다.


## 21. Mailbox Quota

사용자별 메일 저장 공간을 제한하는 Mailbox Quota를 적용할 수 있도록 설계한다.

초기 테스트:

`500 MB / User`

실제 운영 환경에서는 조직 규모와 저장 공간을 고려하여 조정한다.

Quota를 통해 특정 사용자가 서버 저장 공간을 과도하게 사용하는 것을 방지한다.

## 22. Mail Logging

메일 서버에서 발생하는 주요 이벤트를 로그로 관리한다.

주요 로그 대상:

- SMTP Connection
- Authentication
- Mail Delivery
- Mail Rejection
- Relay Attempt
- IMAP Connection
- Authentication Failure

로그를 통해 메일 송수신 문제 및 보안 이벤트를 추적할 수 있도록 한다.


## 23. Authentication Failure Monitoring

반복적인 인증 실패가 발생하는 경우 보안 이벤트로 판단할 수 있도록 로그를 확인한다.

예:

    User: sun0001
    Service: SMTP
    Result: Authentication Failed

반복적인 실패가 발생하면 계정 문제 또는 비정상적인 접근 시도 여부를 확인한다.


## 24. Backup

메일 서버의 사용자 메일함과 설정 파일을 BACKUP01에 백업한다.

Backup Server:

`BACKUP01`

IP:

`10.10.20.30`

기본 구조:

    MAIL01
       |
       | Backup
       ↓
    BACKUP01


## 25. Backup Target

백업 대상:

- Mailbox Data
- Postfix Configuration
- Dovecot Configuration
- TLS Certificate
- User Account Configuration

메일 데이터와 서비스 설정을 함께 백업하여 서버 장애 발생 시 전체 서비스를 복구할 수 있도록 한다.


## 26. Mail Server Failure Scenario

MAIL01에 장애가 발생하는 경우 내부 메일 송수신이 중단될 수 있다.

기본적인 장애 대응:

    MAIL01 Failure
         ↓
    Check Server
         ↓
    Check Postfix
         ↓
    Check Dovecot
         ↓
    Check Storage
         ↓
    Restore Configuration / Mailbox
         ↓
    Mail Service Recovery


## 27. Mail Storage Failure

메일 데이터 저장 영역에 문제가 발생할 경우 메일함 접근 및 메일 송수신에 영향을 줄 수 있다.

따라서 다음 항목을 확인한다.

1. Disk 상태
2. Filesystem 상태
3. Mailbox Directory
4. Storage Capacity
5. Dovecot 상태
6. Backup 상태


## 28. Monitoring Integration

MAIL01은 MON01에서 모니터링한다.

MON01:

`10.10.20.40`

모니터링 대상:

- Server Availability
- CPU
- Memory
- Disk
- Postfix Service
- Dovecot Service
- SMTP Port
- IMAP Port
- Mail Storage Usage


## 29. Security Considerations

메일 서버에는 다음 보안 원칙을 적용한다.

1. Open Relay 차단
2. SMTP 인증 적용
3. TLS 적용
4. 불필요한 Port 차단
5. 관리자 권한 최소화
6. Mailbox Quota 적용
7. 인증 실패 로그 확인
8. Mail Log 관리
9. 정기적인 업데이트
10. 메일 데이터 백업


## 30. Implementation Scope

이번 프로젝트에서는 다음 항목을 실제 구축한다.

- MAIL01
- Linux Server
- Postfix
- Dovecot
- SMTP
- SMTP Submission
- IMAP
- IMAPS
- Internal DNS Record
- Internal Mail Delivery
- User Mailbox
- SMTP Authentication
- Open Relay Prevention
- TLS
- Mailbox Quota
- Mail Logging
- Backup
- Monitoring


## 31. Implementation Order

메일 서버는 다음 순서로 구축한다.

1. MAIL01 VM 생성
2. Linux 설치
3. Static IP 설정
4. DNS 설정
5. Hostname 설정
6. Postfix 설치
7. Postfix 기본 설정
8. SMTP 테스트
9. Dovecot 설치
10. IMAP 설정
11. Mailbox 생성
12. SMTP Authentication 설정
13. Open Relay 차단
14. TLS 설정
15. DNS Record 생성
16. Mail Client 설정
17. Internal Mail Test
18. Authentication Failure Test
19. Mail Log 확인
20. Mailbox Quota 설정
21. Backup 구성
22. Restore Test
23. MON01 Monitoring 연계


## 32. Validation Checklist

### SMTP

- SMTP Service 정상
- Port 25 연결 확인
- Port 587 연결 확인
- 인증된 사용자 발송 가능
- 인증되지 않은 Relay 차단


### IMAP

- Dovecot 정상
- Port 993 연결 확인
- 사용자 Mailbox 접근 가능
- TLS 연결 확인


### DNS

- `mail.suntech.local` 조회 가능
- MAIL01 IP 정상 반환
- Client에서 Mail Server 이름 해석 가능


### Mail

- SUN0001 → SUN0002 발송 성공
- SUN0002 → SUN0001 발송 성공
- Mailbox 수신 확인
- Mail Client에서 메일 조회 가능


### Security

- Open Relay 차단
- SMTP Authentication 정상
- TLS 정상
- 인증 실패 로그 기록
- 불필요한 Port 차단


### Backup

- Mailbox Backup 정상
- Configuration Backup 정상
- Mailbox Restore 정상
- Mail Service 복구 정상


## 33. Final Architecture

SunTech Mail Server는 다음과 같은 구조로 구성한다.

    User PC
       |
       | SMTP 587 / IMAPS 993
       ↓
    MAIL01
    10.10.20.80
       |
       +------------------+
       |                  |
    Postfix            Dovecot
       |                  |
     SMTP               IMAP
       |                  |
       +--------+---------+
                |
           User Mailbox


    Active Directory
           |
           ↓
      User Identity
           |
           ↓
         MAIL01


    Internal DNS
           |
           ↓
    mail.suntech.local
           |
           ↓
      10.10.20.80


    BACKUP01
           |
           ↓
         MAIL01


    MON01
           |
           ↓
         MAIL01


## 34. Final Design Principle

SunTech Mail Server는 단순한 메일 송수신 기능보다 기업 IT 인프라 운영 관점에서 설계한다.

Postfix를 통해 SMTP 메일 전송을 담당하고 Dovecot을 통해 사용자 Mailbox에 대한 IMAP 접근을 제공한다.

Active Directory 및 내부 DNS와 연계하여 사용자 식별과 서버 이름 해석을 지원하고, SMTP Authentication과 Open Relay 방지를 적용하여 메일 서버의 기본적인 보안 수준을 확보한다.

또한 TLS를 적용하여 사용자와 메일 서버 간 통신을 보호하고, Mail Log를 통해 장애 및 보안 이벤트를 추적한다.

MAIL01의 메일 데이터와 설정은 BACKUP01에 백업하며, MON01을 통해 서비스 상태와 저장 공간을 모니터링한다.

이번 프로젝트에서는 내부 메일 송수신 환경을 우선 구축하고, 외부 메일 송수신 및 SPF, DKIM, DMARC 등의 고급 메일 보안 기능은 향후 확장 범위로 관리한다.
