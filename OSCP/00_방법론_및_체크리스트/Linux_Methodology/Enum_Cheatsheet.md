---
layout: page
title: Enum_Cheatsheet
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---


# Linux 권한 상승을 위한 수동 열거 치트시트

이 문서는 자동화된 스크립트(예: linpeas.sh) 사용이 제한되거나, 특정 항목에 대한 심층 분석이 필요할 때 사용하는 핵심 수동 열거 명령어들을 모아 놓은 치트시트입니다.

---

### **1. 시스템 기본 정보 (System Basics)**

- **목표:** 운영체제 버전, 커널 정보, 환경 변수 등 시스템의 기본적인 정보를 파악하여 공격의 기반을 마련합니다.

```bash
# 커널 버전 확인 (-> 커널 익스플로잇 검색에 사용)
uname -a
uname -sr

# 배포판 정보 확인
cat /etc/os-release
lsb_release -a
```

```bash
# PATH, HOME 등 주요 환경 변수 확인 (비정상적인 경로가 있는지 체크)
env
```

---


### **2. 사용자 및 권한 (Users & Permissions)**

- **목표:** 현재 사용자의 권한 수준을 파악하고, `sudo`나 특수 권한(SUID/SGID)을 통해 권한을 상승시킬 수 있는 지점을 찾습니다.

```bash
# 현재 사용자, 그룹, UID, GID 확인
id
```

```bash
# 현재 사용자가 sudo를 통해 어떤 명령어를 어떤 권한으로 실행할 수 있는지 확인
# (NOPASSWD: 키워드가 있다면 비밀번호 없이 실행 가능)
sudo -l
```

```bash
# SUID 비트가 설정된 파일을 검색 (가장 고전적이고 강력한 권한 상승 벡터 중 하나)
# - GTFOBins (https://gtfobins.github.io/) 사이트에서 해당 바이너리로 권한 상승이 가능한지 확인
find / -type f -perm -4000 -ls 2>/dev/null

strings /usr/sbin/exim4
# (Use searchsploit and gtfobins)
```

```bash
# 특정 프로세스에 부여된 세분화된 권한(Capabilities)을 확인
# - 예: cap_net_raw+ep는 패킷 스니핑이 가능함을 의미
getcap -r / 2>/dev/null
```

---


### **3. 프로세스 및 예약 작업 (Processes & Scheduled Tasks)**

- **목표:** 현재 실행 중인 프로세스와 주기적으로 실행되는 작업(Cronjob)을 분석하여, 권한이 높게 실행되는 프로세스나 수정 가능한 스크립트를 찾습니다.

```bash
# 시스템에서 실행 중인 모든 프로세스를 상세히 확인 (특히 root 권한으로 실행되는 특이한 프로세스)
ps aux | grep root
./pspy64
```

```bash
# 시스템 전체 및 사용자별 Cron 작업 목록을 확인
# - 누구나 수정 가능한 스크립트가 root 권한으로 실행되는지 집중적으로 확인
cat /etc/crontab
ls -la /etc/cron.*
cat /var/spool/cron/crontabs/root
cat /etc/crontab
ls /etc/cron''
cat /etc/cron''
cat /etc/cron.d
cat /var/spool/cron/''
```

---

### **4. 네트워크 및 서비스 (Networking & Services)**

- **목표:** 시스템 내부에서만 접근 가능한 서비스(127.0.0.1)를 찾아내어, 외부에서 접근할 수 없었던 새로운 공격 지점을 식별합니다.

```bash
# 현재 시스템에서 리스닝 중인 TCP/UDP 포트 확인
# - 127.0.0.1 (localhost)에서만 실행 중인 서비스를 주목
ss -lntu
ss -ltu
netstat -lntu
```

---


### **5. 파일 시스템 및 중요 파일 (Filesystem & Sensitive Files)**

- **목표:** 모든 사용자가 쓰기 가능한 디렉터리, 설정 파일, 로그 파일, 개인 키 파일 등 민감한 정보가 포함된 파일을 찾습니다.

```bash
# 모든 사용자가 파일을 생성/수정할 수 있는 디렉터리 검색
find / -writable -type d 2>/dev/null
```

```bash
find / -group uwu 2> /dev/null
find / -writable -type d -group gods 2> /dev/null
find / -writable -type f -group gods 2> /dev/null
```

```bash
id
find / -writable -type d -user hades 2> /dev/null
find / -writable -type f -user hades 2> /dev/null
```

```bash
# 다른 사용자의 홈 디렉터리, 특히 .ssh, .bash_history 등 숨겨진 파일 확인
ls -la /home
ls -la /root
```

```bash
# 시스템 및 서비스 로그에서 사용자 정보, 비밀번호 등 민감한 정보가 있는지 확인
cat /var/log/syslog
cat /var/log/auth.log
```

