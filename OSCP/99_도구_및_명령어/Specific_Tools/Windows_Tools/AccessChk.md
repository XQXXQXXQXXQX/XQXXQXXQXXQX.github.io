

# AccessChk 활용 가이드

`AccessChk`는 Sysinternals Suite에 포함된 명령줄 도구로, Windows 시스템의 파일, 디렉터리, 레지스트리 키, 서비스, 프로세스 등 다양한 객체에 대한 유효 권한을 확인하는 데 사용됩니다. 권한 상승 취약점을 찾거나 시스템의 보안 설정을 감사하는 데 필수적인 도구입니다.

---

### **1. 주요 기능 및 옵션**

`AccessChk`는 매우 다양한 객체 유형과 옵션을 지원합니다.

| 옵션 | 설명 |
| :--- | :--- |
| `-u` | 오류 메시지를 표시하지 않습니다. |
| `-w` | 쓰기 권한이 있는 객체만 표시합니다. |
| `-c` | 서비스를 확인합니다. |
| `-q` | 간결한 출력 모드 (배너 표시 안 함). |
| `-v` | 상세 출력 모드 (모든 권한 표시). |
| `-d` | 디렉터리 권한만 표시하고 파일 권한은 표시하지 않습니다. |
| `-s` | 하위 디렉터리까지 재귀적으로 확인합니다. |
| `-accepteula` | EULA(최종 사용자 라이선스 계약)를 자동으로 수락합니다. |

---

### **2. 활용 시나리오**

#### **서비스 권한 확인**
- **목표:** 특정 서비스 또는 모든 서비스에 대해 현재 사용자나 특정 그룹이 수정 권한을 가지고 있는지 확인합니다. 이는 `취약한 서비스 권한` 공격에 활용됩니다.

```powershell(title="특정 서비스 권한 확인")
# daclsvc 서비스에 대한 모든 권한 확인
accesschk.exe -ucqv daclsvc
```

```powershell(title="모든 서비스에 대한 쓰기 권한 확인")
# Authenticated Users 그룹이 쓰기 권한을 가진 모든 서비스를 검색
accesschk.exe -ucqv "Authenticated Users" *
```

#### **파일/디렉터리 쓰기 권한 확인**
- **목표:** 특정 파일이나 디렉터리에 대해 현재 사용자가 쓰기 권한을 가지고 있는지 확인합니다. 이는 `따옴표 없는 서비스 경로` 공격이나 파일 업로드 취약점 등에 활용됩니다.

```powershell(title="디렉터리 쓰기 권한 확인")
# C:\Program Files\Unquoted Path Service\ 디렉터리에 대한 쓰기 권한 확인
accesschk.exe /accepteula -uwqd "C:\Program Files\Unquoted Path Service\"
```

```powershell(title="특정 경로 하위의 모든 쓰기 가능한 파일/폴더 검색")
# C:\ 드라이브 하위에서 Authenticated Users가 쓰기 가능한 모든 파일/폴더 검색
accesschk.exe -uwv "Authenticated Users" C:\ 
```

#### **레지스트리 권한 확인**
- **목표:** 특정 레지스트리 키에 대한 권한을 확인하여 레지스트리 기반의 권한 상승 취약점을 찾습니다.

```powershell(title="레지스트리 키 권한 확인")
# HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run 키에 대한 권한 확인
accesschk.exe -ukv HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

---

### **3. 다운로드 및 실행**

- **다운로드:** [Microsoft Sysinternals 웹사이트](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk)에서 `AccessChk`를 다운로드할 수 있습니다.
- **실행:** 다운로드한 `accesschk.exe` (또는 `accesschk64.exe`)를 타겟 시스템에 업로드한 후 명령 프롬프트나 PowerShell에서 실행합니다.

```powershell(title="AccessChk 실행 예시")
# EULA를 자동으로 수락하고 실행
.
accesschk64.exe -accepteula -ucqv daclsvc
```

