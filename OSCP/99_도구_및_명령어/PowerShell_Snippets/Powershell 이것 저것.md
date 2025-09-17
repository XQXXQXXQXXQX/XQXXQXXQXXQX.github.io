


# PowerShell 기본 열거 및 파일 검색 스니펫

PowerShell은 Windows 시스템 관리 및 침투 후 정보 수집(Post-Exploitation Enumeration)에 매우 강력한 도구입니다. 별도의 바이너리 업로드 없이 기본적으로 제공되는 기능만으로도 시스템의 다양한 정보를 효과적으로 수집할 수 있습니다.

---

### **1. 시스템 기본 정보 열거**

- **목표:** 운영체제, 컴퓨터 이름, 도메인/워크그룹, 시스템 아키텍처 등 기본적인 시스템 정보를 파악합니다.

```powershell(title="컴퓨터 정보 요약")
Get-ComputerInfo
```

```powershell(title="OS 및 시스템 상세 정보")
Get-WmiObject Win32_OperatingSystem
Get-WmiObject Win32_ComputerSystem
```

---

### **2. 사용자 및 그룹 정보**

- **목표:** 로컬 사용자 계정, 그룹, 그리고 각 그룹의 멤버십을 확인하여 권한 상승 가능성을 탐색합니다.

```powershell(title="로컬 사용자 및 그룹")
Get-LocalUser
Get-LocalGroup
```

```powershell(title="특정 그룹 멤버 확인")
# Administrators 그룹의 멤버 확인
Get-LocalGroupMember -Group Administrators
```

---

### **3. 파일 시스템 검색**

- **목표:** 특정 파일(예: 플래그 파일), 민감한 설정 파일, 쓰기 가능한 디렉터리 등을 찾아냅니다.

```powershell(title="특정 파일 검색 (예: 플래그)")
# C:\Users 경로에서 .txt 파일을 재귀적으로 검색하고, 'flag' 문자열을 포함하는 라인 출력
Get-ChildItem -Path "C:\Users" -Recurse -Include *.txt -ErrorAction SilentlyContinue | Select-String -Pattern "flag" -CaseSensitive
```

```powershell(title="민감한 파일 검색")
# C:\ 드라이브에서 .config, .pass, .cred 확장자를 가진 파일을 검색
Get-ChildItem -Path "C:\" -Recurse -Include *.config, *pass*, *cred* -ErrorAction SilentlyContinue
```

```powershell(title="쓰기 가능한 디렉터리 검색 (고급)")
# Authenticated Users 그룹이 쓰기 권한을 가진 디렉터리 검색
Get-ChildItem -Path "C:\" -Recurse -Directory -ErrorAction SilentlyContinue | Where-Object { ($_.FullName | Get-Acl).Access | Where-Object { $_.IdentityReference -like "*Authenticated Users*" -and $_.FileSystemRights -match "Write" } }
```

```powershell title="Users 폴더 하위에 .txt 찾기"
Get-ChildItem -Path "C:\Users" -Recurse -Filter *.txt | Get-Content
```

---

### **4. 프로세스 및 서비스 정보**

- **목표:** 현재 실행 중인 프로세스와 설치된 서비스를 확인하여 비정상적인 프로세스나 취약한 서비스를 식별합니다.

```powershell(title="실행 중인 프로세스")
Get-Process
```

```powershell(title="설치된 서비스")
GetService
GetService | Where-Object {$_.Status -eq "Running"} # 실행 중인 서비스만
```

---

### **5. 네트워크 정보**

- **목표:** IP 주소, 라우팅 테이블, 활성 네트워크 연결, 방화벽 규칙 등을 확인하여 네트워크 환경을 파악합니다.

```powershell(title="IP 주소 및 네트워크 인터페이스")
Get-NetIPAddress
```

```powershell(title="라우팅 테이블")
Get-NetRoute
```

```powershell(title="활성 TCP 연결")
Get-NetTCPConnection
```

```powershell(title="방화벽 규칙")
Get-NetFirewallRule
```

