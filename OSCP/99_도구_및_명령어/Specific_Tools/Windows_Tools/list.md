


# Windows 모의 해킹 도구 모음

이 문서는 Windows 환경에서 모의 해킹 및 침투 테스트에 사용되는 주요 도구들을 기능별로 분류하고, 각 도구에 대한 간략한 설명과 함께 상세 가이드 링크를 제공합니다.

---

### **1. 정보 수집 및 열거 (Information Gathering & Enumeration)**

-   **AccessChk**
    -   **설명:** 파일, 디렉터리, 레지스트리 키, 서비스 등 다양한 객체에 대한 유효 권한을 확인합니다.
    -   **[상세 가이드](AccessChk.md)**

-   **WinPEAS (Windows Privilege Escalation Awesome Script)**
    -   **설명:** Windows 시스템의 권한 상승 벡터를 종합적으로 분석하는 스크립트입니다.
    -   **[상세 가이드](../PEASS.md)**

-   **Sherlock.ps1**
    -   **설명:** Windows 시스템에 누락된 보안 패치를 찾아내어 커널 익스플로잇 가능성을 식별합니다.
    -   **[GitHub 링크](https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1)**

---

### **2. 권한 상승 (Privilege Escalation)**

-   **PowerUp.ps1**
    -   **설명:** 서비스 설정 오류, 따옴표 없는 서비스 경로 등 Windows의 흔한 권한 상승 취약점을 자동으로 찾아냅니다.
    -   **[상세 가이드](PowerUp.ps1.md)**

---

### **3. 크리덴셜 덤프 (Credential Dumping)**

-   **Mimikatz**
    -   **설명:** Windows 메모리에서 평문 비밀번호, 해시, Kerberos 티켓 등을 추출하는 강력한 도구입니다.
    -   **[상세 가이드](OSCP/99_도구_및_명령어/Specific_Tools/Windows_Tools/Mimikatz.md)**

-   **pwdump.py**
    -   **설명:** SAM(Security Account Manager) 데이터베이스에서 로컬 사용자 계정의 NTLM 해시를 추출합니다.
    -   **[상세 가이드](creddump7.md)**

---

### **4. 원격 실행 및 측면 이동 (Remote Execution & Lateral Movement)**

-   **Impacket Suite**
    -   **설명:** `psexec.py`, `wmiexec.py` 등 SMB, WMI, RPC를 이용한 원격 실행 및 다양한 AD 공격 스크립트를 포함합니다.
    -   **[상세 가이드](../../Impacket 주요 도구 활용 가이드.md)**

-   **Evil-WinRM**
    -   **설명:** WinRM을 통해 Windows 시스템에 PowerShell 셸을 획득하고 파일 전송 등 다양한 작업을 수행합니다.
    -   **[상세 가이드](../../../01_정찰_및_정보_수집/User_Discovery/user discovery/evil-winrm.md)**

-   **SharpGPOAbuse**
    -   **설명:** GPO(그룹 정책 개체)의 잘못된 설정을 악용하여 권한 상승 또는 지속성을 확보합니다.
    -   **[상세 가이드](SharpGPOAbuse.md)**

---

### **5. 파일 전송 및 네트워크 도구**

-   **Ncat (Netcat)**
    -   **설명:** Windows 환경에서 리버스/바인드 셸, 파일 전송, 포트 스캐닝 등 다양한 네트워크 작업을 수행합니다.
    -   **[상세 가이드](../../Shell_and_Payloads/Windows_Netcat.md)**

-   **정적 바이너리 (Static Binaries)**
    -   **설명:** 외부 DLL 의존성 없이 자체적으로 실행 가능한 도구들로, 이식성과 스텔스 측면에서 유리합니다.
    -   **[상세 가이드](ETC  (binaries).md)**












PowerUp.ps1
https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerUp/PowerUp.ps1


AccessChk
https://learn.microsoft.com/ko-kr/sysinternals/downloads/accesschk


service-acl.ps1
https://rohnspowershellblog.wordpress.com/2013/03/19/viewing-service-acls/


winPEASx64.exe
https://github.com/peass-ng/PEASS-ng/releases/tag/20250701-bdcab634


Sherlock.ps1
https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1


