---
layout: page
title: Check_List_(AD)
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---


### **Active Directory (액티브 디렉터리):**

- [ ] 모든 TCP 포트 스캔

- [ ] 익명 접속(anonymous access)으로 ldap, rpc, smb 확인. 공개 공유(public share) 확인.

- [ ] 사용자 이름 찾기: 접근 권한을 얻었다면, `netexec`의 `--rid-brute`, `--users` 옵션이나 `rpcclient`의 `enumdomuser` 같은 도구를 사용하여 초기 인증 정보 없이 도메인 사용자 목록을 획득합니다. `kerbrute`를 사용하여 사용자 이름 유효성을 확인합니다.

- [ ] 사용자 이름을 수집한 후 AS-REP Roasting 공격을 테스트합니다. 성공하면 크래킹을 시도합니다.

- [ ] 사용자 이름과 암호를 확보한 후 Kerberoasting 공격을 확인합니다.

- [ ] 확보한 인증 정보 세트로 가능한 모든 프로토콜(winrm, rdp, mssql, smb, rpc, ldap 등)에 인증을 시도합니다.

- [ ] 접근 가능한 모든 사용자의 공유 폴더를 열거(Enumerate)합니다. 새로운 사용자를 발견할 때마다 해당 사용자의 공유 폴더를 다시 확인해야 합니다.

- [ ] 사용자 권한으로 쉘을 얻었다면, 권한 상승(privesc)을 확인하고, 모든 해시를 덤프하여 파일로 수집합니다. 이를 통해 무차별 대입 공격(bruteforcing)이나 측면 이동(lateral movement)을 시도할 수 있습니다.

- [ ] BloodHound를 실행하여 DCSync와 같은 기본 공격 경로를 확인합니다. BloodHound가 찾아낼 수 있는 다른 공격 경로도 확인합니다.

- [ ] AD 네트워크 환경이라면, 피버팅(pivoting)이 필요할 경우를 대비해 추가 네트워크 어댑터가 있는지 확인합니다. 만약 있다면, 해당 "점프박스(jumpbox)"를 통해 피벗을 설정하고 모든 도구를 릴레이(relay)하여 다른 네트워크의 머신들을 공격 목표로 삼습니다.

- [ ] 보유한 인증 정보로 여러 다른 프로토콜에 무차별 대입 공격을 시도하는 것을 기억하세요. NTLM 인증 또한 사용하는 것을 잊지 마세요.

- [ ] UDP 포트를 확인합니다.

- [ ] 진행이 막혔다면 재스캔하고, 도구들이 정상적으로 작동하는지, 그리고 올바르게 실행하고 있는지 확인하세요.


---

### **Windows Privesc (윈도우 권한 상승):**

- [ ] `whoami /all` 실행 후, PowerUp, 그 다음 WinPEAS를 실행합니다.

- [ ] 이 과정에서 거의 모든 정보를 얻을 수 있습니다. 출력 결과를 저장하고 천천히 검토하세요.

- [ ] 만약 완전히 막혔다면, 수동으로 여러 항목을 직접 확인할 수도 있습니다. 그래도 도움이 되지 않는다면, 권한 상승 공격 기법들을 다시 학습해야 할 가능성이 높습니다.


