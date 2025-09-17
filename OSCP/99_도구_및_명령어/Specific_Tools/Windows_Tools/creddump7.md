
# pwdump.py 활용 가이드

`pwdump.py`는 Windows 시스템의 SAM(Security Account Manager) 데이터베이스에서 로컬 사용자 계정의 NTLM 해시를 추출하는 Python 스크립트입니다. 이는 `Pass-the-Hash` 공격이나 오프라인 해시 크래킹을 위한 필수적인 정보 수집 단계입니다.

---

### **1. 작동 원리**

`pwdump.py`는 Windows 레지스트리의 `SAM` 및 `SYSTEM` 하이브 파일에 접근하여 사용자 계정 정보와 암호화된 해시 값을 읽어옵니다. `SYSTEM` 하이브에는 `SAM` 하이브의 내용을 복호화하는 데 필요한 부팅 키(Boot Key)가 포함되어 있습니다.

- **필요 권한:** 일반적으로 시스템의 관리자 권한이 필요합니다.

---

### **2. 다운로드 및 실행**

`pwdump.py`는 Impacket 프레임워크의 예제 스크립트 중 하나로 포함되어 있습니다. Impacket 설치 시 함께 제공됩니다.

#### **활성 시스템에서 직접 실행**
- 타겟 시스템에 Python과 Impacket이 설치되어 있고, 관리자 권한의 셸을 획득한 경우 직접 실행할 수 있습니다.

```bash(title="pwdump.py 직접 실행")
python2 pwdump.py <target_ip> <username> <password>
```

#### **오프라인 분석 (권장)**
- `SAM` 및 `SYSTEM` 하이브 파일을 먼저 탈취한 후, 공격자 머신에서 `pwdump.py`를 사용하여 해시를 추출하는 것이 더 일반적이고 스텔스한 방법입니다. (하이브 파일 탈취 방법은 `Windows 로컬 크리덴셜 덤프 가이드` 참고)

```bash(title="pwdump.py 오프라인 분석")
# -s: SAM 하이브 파일 경로
# -y: SYSTEM 하이브 파일 경로
python2 pwdump.py -s C:\Temp\SAM -y C:\Temp\SYSTEM
```

---

### **3. 해시 활용**

`pwdump.py`를 통해 추출된 NTLM 해시는 다음과 같은 후속 공격에 활용될 수 있습니다.

-   **Pass-the-Hash (PtH) 공격:** 평문 비밀번호를 알지 못해도 해시만으로 다른 시스템에 인증하여 측면 이동(Lateral Movement)을 수행합니다. (자세한 내용은 `Impacket 주요 도구 활용 가이드` 참고)

-   **오프라인 크래킹:** `hashcat`이나 `John the Ripper`와 같은 도구를 사용하여 해시를 크랙하고 평문 비밀번호를 복구합니다. 이는 추가적인 공격 기회를 제공할 수 있습니다.

```bash(title="Hashcat으로 NTLM 해시 크래킹")
hashcat -m 1000 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

