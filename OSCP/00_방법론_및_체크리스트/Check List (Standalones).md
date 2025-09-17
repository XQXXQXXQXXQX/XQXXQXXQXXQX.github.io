
#### **Initial Recon & Access (초기 정찰 및 접근)**

- [ ] Scan all TCP ports. (모든 TCP 포트 스캔)

- [ ] Enumerate web server with all your relevant wordlists. (관련 단어 목록을 사용하여 웹 서버 정보 수집)

- [ ] Do proper fingerprinting with -sV -sC, nmap vuln scan, wappalyzer, check CMS, check server headers, nc -nvv, check every TCP port etc. (nmap -sV -sC, 취약점 스캔, wappalyzer, CMS 확인, 서버 헤더 확인, nc -nvv 등 모든 TCP 포트를 확인하여 정확한 핑거프린팅 수행)

- [ ] Check for public exploits on exploitDB and google. (ExploitDB와 구글에서 공개된 익스플로잇 검색)

- [ ] Rescan if you're stuck, verify tools are working properly and you're running them properly. (진행이 막히면 재스캔하고, 도구가 올바르게 작동하고 실행되는지 확인)

- [ ] Check UDP ports. (UDP 포트 확인)

#### **Linux Privesc (리눅스 권한 상승)**

- [ ] Run Linpeas & Possible LSE(Linux Smart Enumeration) (Linpeas 및 LSE 실행)

- [ ] This will give you all info tbh. Save output and slowly go over it. (이 도구들이 모든 정보를 줄 것입니다. 결과를 저장하고 천천히 검토하세요.)

- [ ] If you get completely stuck, you can manually check for things too. If that doesn't help you likely need to relearn the privesc attacks. (완전히 막혔다면 수동으로 확인할 수도 있습니다. 그래도 해결되지 않으면 권한 상승 공격에 대해 다시 학습해야 할 것입니다.)


#### **Windows Privesc (윈도우 권한 상승)**

- [ ] Run whoami /all, then PowerUp, then WinPEAS (whoami /all 실행 후, PowerUp, 그 다음 WinPEAS 실행)

- [ ] This will give you all info tbh. Save output and slowly go over it. (이 도구들이 모든 정보를 줄 것입니다. 결과를 저장하고 천천히 검토하세요.)

- [ ] If you get completely stuck, you can manually check for things too. If that doesn't help you likely need to relearn the privesc attacks. (완전히 막혔다면 수동으로 확인할 수도 있습니다. 그래도 해결되지 않으면 권한 상승 공격에 대해 다시 학습해야 할 것입니다.)
