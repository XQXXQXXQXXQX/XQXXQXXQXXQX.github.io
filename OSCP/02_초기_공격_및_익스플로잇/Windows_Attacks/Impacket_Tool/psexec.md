---
layout: page
title: psexec
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# Impacket-psexec 원격 실행 가이드

`impacket-psexec`은 원격 Windows 시스템에서 대화형 셸(Interactive Shell)을 획득하는 데 사용되는 강력한 도구입니다. SMB 프로토콜(445/TCP)을 통해 작동하며, 시스템 관리자 권한이 있을 때 주로 사용됩니다.

---

### **1. 기본 접속 방법**

- **목표:** 획득한 자격증명을 사용하여 대상 시스템의 `cmd.exe` 셸을 획득합니다.
- **팁:** `rlwrap`과 함께 사용하면 화살표 키를 통한 명령어 히스토리 조회가 가능해져 훨씬 편리합니다.

```bash
# -u: 사용자 이름
# -p: 비밀번호
# @<target>: 타겟 IP 주소 또는 호스트명
rlwrap impacket-psexec <domain>/<user>:<password>@$target
```

```bash
# -hashes: NTLM 해시 (LM해시는 비워둠)
rlwrap impacket-psexec <domain>/<user>@$target -hashes :<ntlm_hash>
```

---

### **2. 작동 원리 및 탐지**

`psexec`은 다음과 같은 원리로 동작하며, 이 때문에 최신 보안 솔루션에 의해 탐지될 가능성이 있습니다.

1.  **파일 업로드:** `ADMIN
라는 관리 공유 폴더를 통해 `PSEXESVC.exe`라는 악성 서비스 파일을 원격 시스템에 업로드합니다.
2.  **서비스 생성:** 원격으로 Windows 서비스 제어 관리자(SCM)를 조작하여 업로드된 `PSEXESVC.exe`를 실행하는 새로운 서비스를 생성하고 시작합니다.
3.  **파이프 통신:** 생성된 서비스와 명명된 파이프(Named Pipe)를 통해 통신하며 명령어를 주고받습니다.

> 이처럼 파일을 디스크에 쓰고 서비스를 생성하는 행위는 최신 EDR이나 백신(AV) 솔루션에 의해 악의적인 행위로 탐지 및 차단될 수 있습니다.

---

### **3. 다른 원격 실행 도구와의 비교**

| 도구             | 프로토콜          | 특징               | 장점                             | 단점                         |
| :------------- | :------------ | :--------------- | :----------------------------- | :------------------------- |
| **psexec.py**  | SMB (445)     | 원격 서비스 생성        | 가장 일반적이고 안정적                   | 탐지 가능성 높음                  |
| **wmiexec.py** | RPC/WMI (135) | WMI를 통해 명령어 실행   | 파일리스(Fileless), 더 은밀함          | 셸이 다소 불안정할 수 있음            |
| **evil-winrm** | WinRM (5985)  | PowerShell 원격 관리 | 완전한 기능의 PowerShell 셸, 파일 전송 용이 | 타겟에 WinRM 서비스가 활성화되어 있어야 함 |

**결론:** 초기 침투 시도 시, 탐지를 피하기 위해 `wmiexec`이나 `evil-winrm`을 먼저 시도해보고, 실패할 경우 `psexec`을 사용하는 것이 일반적인 전략입니다.


