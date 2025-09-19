---
layout: page
title: AllowedToDelegate
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

# 제한 없는 위임 (Unconstrained Delegation) 공격 가이드

제한 없는 위임(Unconstrained Delegation)은 오래된 위임 방식으로, 특정 서버가 자신에게 인증한 사용자를 대신하여 **네트워크상의 어떤 서비스에도** 접근할 수 있도록 허용하는 매우 위험한 설정입니다. 공격자가 이 설정이 활성화된 서버의 관리자 권한을 획득하면, 해당 서버에 접속하는 다른 사용자(특히 도메인 관리자)의 권한을 완전히 탈취할 수 있습니다.

### **공격 원리: TGT 탈취**

1.  사용자가 제한 없는 위임이 설정된 서버(서버 A)에 Kerberos 인증을 시도합니다.
2.  인증 과정에서 사용자의 TGT(Ticket-Granting Ticket)가 서버 A의 LSASS 메모리에 저장됩니다.
3.  공격자는 서버 A의 관리자 권한으로 LSASS 메모리를 덤프하여 이 TGT를 훔칩니다.
4.  훔친 TGT를 자신의 세션에 주입(Pass-the-Ticket)하면, 공격자는 해당 사용자의 권한으로 도메인 내의 다른 모든 서비스(예: 도메인 컨트롤러)에 접근할 수 있게 됩니다.

---

BloodHound 이용해서 식별 후 GetST로 진행해도 됨.
![GetST]

![Pasted_image_20250804233330.png](/image/Pasted_image_20250804233330.png)

---

### **1. 취약점 식별**

- **목표:** `TRUSTED_FOR_DELEGATION` 속성이 활성화된 컴퓨터 또는 사용자 계정을 찾습니다.

#### **BloodHound 활용**
- BloodHound에서 "Find Computers with Unconstrained Delegation" 쿼리를 실행하는 것이 가장 직관적이고 효과적인 방법입니다.

#### **PowerView를 이용한 수동 열거**

```powershell
# 제한 없는 위임이 설정된 컴퓨터 계정 검색
Get-NetComputer -Unconstrained

# 제한 없는 위임이 설정된 사용자 계정 검색
Get-NetUser -Unconstrained
```

---

### **2. 단계별 공격 시나리오**

1.  **서버 장악:** 제한 없는 위임이 설정된 서버의 로컬 관리자 권한을 획득합니다.

2.  **관리자 인증 대기:** 도메인 관리자(DA)가 해당 서버에 어떤 방식으로든 인증하기를 기다립니다. (예: 원격 데스크톱, 파일 공유 접속, 원격 관리 등)

3.  **TGT 덤프:** DA가 인증하면, 서버의 LSASS 메모리에 DA의 TGT가 저장됩니다. `mimikatz`를 사용하여 메모리에 캐시된 모든 Kerberos 티켓을 덤프합니다.
    ```powershell
    # Mimikatz 실행 후
    privilege::debug
    sekurlsa::tickets /export
    ```
    - 이 명령은 현재 디렉터리에 `.kirbi` 확장자를 가진 여러 티켓 파일을 생성합니다.

4.  **DA 티켓 식별:** 생성된 `.kirbi` 파일 중 도메인 관리자 계정의 티켓을 찾습니다. (파일 이름에 사용자 이름이 포함됨)

5.  **Pass-the-Ticket (PTT) 공격:** 식별된 DA의 티켓을 현재 세션에 주입합니다.
    ```powershell
    kerberos::ptt C:\Temp\\\[...]-Administrator@<domain>-.kirbi
    ```

6.  **권한 확인:** 티켓 주입 후, `klist` 명령어로 티켓이 세션에 로드되었는지 확인합니다. 이제 현재 세션은 DA의 권한을 가지므로, 도메인 컨트롤러의 C$ 공유 폴더 등에 접근할 수 있습니다.
    ```powershell
    klist
    dir \\<DC_IP>\C$
    ```

---

### **3. 방어 (Mitigation)**

- **제한된 위임 사용:** 가능하면 항상 리소스 기반 제한된 위임(RBCD)을 사용하고, 레거시 시스템이 아니라면 제한 없는 위임은 비활성화합니다.
- **중요 계정 보호:** 도메인 관리자 등 중요 계정의 속성에서 "계정은 민감하므로 위임할 수 없음(Account is sensitive and cannot be delegated)" 옵션을 활성화합니다.
- **Admin Tier 모델 도입:** 관리자 계정이 하위 등급의 서버나 워크스테이션에 로그인하지 않도록 하여 TGT 노출 위험을 줄입니다.










[[GetST]]

![[Pasted image 20250804233330.png]]

