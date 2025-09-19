---
layout: page
title: Network_Scanning
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# 네트워크 스캔 가이드 (Nmap 활용)

네트워크 스캔은 모의 해킹의 첫 단추를 끼우는 과정입니다. 이 단계의 목표는 타겟 시스템의 "문(Port)"들이 얼마나 열려 있는지, 그리고 그 문 뒤에는 어떤 "서비스"가 실행되고 있는지를 파악하는 것입니다. Nmap은 이 작업을 수행하는 가장 강력하고 표준적인 도구입니다.

---

### **1. 스캔 전략: 2단계 접근법**

효율적인 스캔을 위해 일반적으로 2단계 접근법을 사용합니다.

1.  **빠른 포트 식별 (Quick Port Discovery):** 먼저 모든 65,535개 포트를 대상으로 어떤 포트가 열려있는지만 빠르게 확인합니다. 이 단계에서는 속도가 중요합니다.
2.  **상세 정보 스캔 (Detailed Scan):** 1단계에서 식별된 열린 포트들만을 대상으로, 서비스 버전 정보, 기본 스크립트 실행, 취약점 스캔 등 정밀하고 상세한 스캔을 수행합니다.

---

### **2. 필수 Nmap 옵션 해설**

| 옵션                 | 설명                                                      |
| :----------------- | :------------------------------------------------------ |
| `-p-`              | 1번부터 65,535번까지 모든 TCP 포트를 스캔합니다.                        |
| `-sV`              | 포트에서 실행 중인 서비스의 이름과 버전 정보를 파악합니다. (매우 중요)               |
| `-sC`              | `--script=default`와 동일. 해당 서비스에 대한 기본적인 진단 스크립트를 실행합니다. |
| `-Pn`              | Ping을 보내 호스트의 활성 상태를 확인하는 단계를 건너뜁니다. (방화벽 환경에서 필수)      |
| `-oN <file>`       | 결과를 Normal 형식으로 지정된 파일에 저장합니다.                          |
| `-T4`, `-T5`       | 스캔 속도 템플릿. 숫자가 높을수록 빠르지만, 탐지될 위험도 커집니다.                 |
| `--min-rate <num>` | 초당 최소 <num>개의 패킷을 보내도록 속도를 조절합니다. (속도 확보에 유용)           |

---

### **3. 상황별 Nmap 명령어 예시**


```bash
nmap -p- -Pn $target -sV -v --min-rate 1500 --max-rtt-timeout 500ms --max-retries 5 --open -oN nmap_ports.txt
```

```bash
nmap -Pn $target -sC -sV -v -T4 --min-rate 1500 --max-rtt-timeout 500ms --max-retries 3
```



```bash
# 속도를 우선하여 모든 TCP 포트를 빠르게 스캔하고, 열린 포트만 `initial_scan.txt`에 저장
nmap -p- -Pn --min-rate 2000 -T4 -oN initial_scan.txt $target
```

```bash
# 1단계에서 찾은 포트들(예: 22, 80, 445)을 대상으로 상세 스캔 실행
nmap -p 22,80,445 -sV -sC -oN detailed_scan.txt $target
```

```bash
# Rustscan으로 포트를 초고속으로 찾고, 그 결과를 바로 Nmap에 넘겨 상세 스캔을 진행
rustscan -a $target --ulimit 5000 -- -sV -sC -oN nmap_full.txt
```

```bash
# 식별된 서비스를 대상으로 알려진 취약점을 찾는 `vuln` 카테고리의 모든 스크립트를 실행
nmap -p 22,80,445 --script vuln -oN vuln_scan.txt $target
```

---

### **4. Proxychains 연동 스캔 (Pivoting 환경)**

내부망으로 피벗(Pivot)한 상황에서는 `proxychains`를 통해 Nmap 스캔을 수행해야 합니다. 이때 몇 가지 중요한 제약사항이 있습니다.

- **`-sT` (TCP Connect 스캔) 사용 필수:** `proxychains`는 완전한 TCP 연결(3-way handshake)만 중계할 수 있습니다. 따라서 SYN 패킷만 보내는 `-sS` (Stealth 스캔)은 동작하지 않으므로, 반드시 `-sT` 옵션을 사용해야 합니다.
- **`-Pn` (Ping 스킵) 사용 필수:** `proxychains`는 ICMP(Ping) 패킷을 터널링할 수 없으므로, `-Pn` 옵션으로 호스트 활성 확인 단계를 건너뛰어야 합니다.

```bash
# -sT와 -Pn 옵션을 필수로 사용하여 내부망의 다른 호스트를 스캔
proxychains -q nmap -sT -Pn -p 22,80,445 $internal_target_ip
```

