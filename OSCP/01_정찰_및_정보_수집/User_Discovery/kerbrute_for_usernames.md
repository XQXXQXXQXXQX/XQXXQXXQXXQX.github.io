---
layout: page
title: kerbrute_for_usernames
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# Kerbrute 사용자 열거 가이드

`Kerbrute`는 Active Directory 환경에서 Kerberos 프로토콜을 통해 안전하고 빠르게 유효한 사용자 이름을 열거(Enumeration)하는 데 특화된 도구입니다.

- **[GitHub 링크](https://github.com/ropnop/kerbrute)**

### **핵심 기능 및 장점**
- **안전성:** 사용자 계정 잠금(Account Lockout)을 유발하지 않고 사용자 존재 유무를 확인할 수 있습니다.
- **속도:** 여러 스레드를 사용하여 대규모 사용자 목록을 빠르게 테스트할 수 있습니다.
- **정확성:** Kerberos 사전 인증(Pre-Authentication) 응답의 차이를 기반으로 동작하여 매우 정확합니다.

---

### **1. 설치 방법**

가장 간단한 방법은 미리 컴파일된 바이너리를 다운로드하는 것입니다.

```bash
# 최신 릴리즈에서 자신의 OS에 맞는 바이너리 다운로드
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64

# 실행 권한 부여 및 경로 이동
chmod +x kerbrute_linux_amd64
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute

# 확인
kerbrute -h
```

---

### **2. 사용자 이름 열거 (`userenum`)

- **목표:** 준비된 사용자 이름 목록(Wordlist)을 기반으로 도메인에 실제 존재하는 계정을 찾아냅니다.

```bash
# --dc: 도메인 컨트롤러의 IP 주소
# -d: 타겟 도메인 이름 (FQDN)
# users.txt: 테스트할 사용자 이름 목록 파일
kerbrute userenum --dc <DC_IP> -d <domain.local> users.txt
```

- **결과 분석:**
  - `[+] VALID USERNAME:`: 도메인에 존재하는 유효한 사용자 이름입니다.
  - `[*] VALID USERNAME (NO PREAUTH):`: AS-REP Roasting 공격에 취약할 가능성이 있는 사용자입니다.

---

### **3. 작동 원리: Kerberos 사전 인증**

`kerbrute`의 `userenum`은 Kerberos 인증의 첫 단계(AS-REQ)를 이용합니다.

1.  **AS-REQ 전송:** `kerbrute`는 목록의 각 사용자에 대해 TGT(Ticket-Granting Ticket)를 요청하는 AS-REQ 패킷을 KDC(도메인 컨트롤러)에 보냅니다.
2.  **KDC 응답 확인:**
    - **사용자가 존재할 경우:** KDC는 "사전 인증이 필요하다"는 의미의 `KDC_ERR_PREAUTH_REQUIRED` 오류를 응답합니다. `kerbrute`는 이 응답을 보고 해당 사용자가 유효하다고 판단합니다.
    - **사용자가 존재하지 않을 경우:** KDC는 "그런 사용자는 모른다"는 의미의 `KDC_ERR_C_PRINCIPAL_UNKNOWN` 오류를 응답합니다.

이처럼 실제 로그인 시도를 하는 것이 아니라 KDC의 오류 응답 코드 차이만을 이용하기 때문에, 계정 잠금 정책에 영향을 주지 않고 안전하게 사용자 존재 유무를 파악할 수 있습니다.


