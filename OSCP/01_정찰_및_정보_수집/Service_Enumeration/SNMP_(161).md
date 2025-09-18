---
layout: page
title: SNMP_(161)
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---


# SNMP (161) 가이드

SNMP(Simple Network Management Protocol)는 라우터, 스위치, 서버 등 네트워크 장비를 관리하고 모니터링하기 위한 프로토콜입니다. 보안 설정이 미흡한 경우, 시스템의 상세 정보, 사용자 계정, 실행 중인 프로세스 등 민감한 정보가 대량으로 노출될 수 있습니다.

### **핵심 개념**
- **Community String:** SNMP 장비에 접근하기 위한 일종의 비밀번호입니다. `public`(읽기 전용), `private`(읽기/쓰기)가 기본값으로 흔히 사용됩니다.
- **MIB (Management Information Base):** 장비가 관리하는 정보의 계층적인 데이터베이스입니다. 이 정보들은 트리 구조로 구성됩니다.
- SNMP에 저장된 정보는 "트리" 형식이며 왼쪽에서 오른쪽으로 읽음. 예를들어 1.3.2 라면 다음 Diagram과 같음.  [[SNMP Example Diagram]]
- **OID (Object Identifier):** MIB 트리 내에서 각 정보 항목을 가리키는 고유한 숫자 주소입니다. (예: `1.3.6.1.2.1.1.1.0`은 시스템 설명을 의미)

---

### **1. Community String 찾기**

- **목표:** SNMP 장비에 접근하기 위한 유효한 커뮤니티 문자열을 찾습니다.
- **전략:** 널리 사용되는 커뮤니티 문자열 목록을 기반으로 무차별 대입 공격(Brute-force)을 수행합니다.

```bash
# -c: 커뮤니티 문자열 목록 파일 지정
onesixtyone $target -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt

onesixtyone $target -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt
```

---

### **2. 시스템 정보 열거**

- **목표:** 획득한 커뮤니티 문자열을 사용하여 `snmpwalk` 또는 `snmp-check`로 시스템 정보를 수집합니다.

#### **`snmpwalk` 활용**
`snmpwalk`는 지정된 OID부터 시작하여 MIB 트리의 모든 하위 정보를 순차적으로 보여줍니다.

```bash
# -c: 커뮤니티 문자열
# -v1: SNMP 버전 1 (가장 일반적)
# OID를 지정하지 않으면 전체 MIB 정보를 덤프
snmpwalk -c public -v1 $target > snmp_full_walk.txt
```

출력량이 방대하므로, 특정 정보(사용자, 프로세스 등)를 나타내는 알려진 OID를 직접 조회하는 것이 더 효율적입니다.

```bash
# LAN Manager 사용자를 나타내는 OID 조회
snmpwalk -c public -v1 $target 1.3.6.1.4.1.77.1.2.25

snmpwalk -c 'openview' -v1 $target 1.3.6.1.4.1.77.1.2.25 | grep "STRING:" | awk -F '"' '{print $2}' > users.txt
```

#### **`snmp-check` 활용**
`snmp-check`는 `snmpwalk`를 더 사용하기 쉽게 만든 래퍼(Wrapper) 스크립트로, 주요 정보를 자동으로 열거하고 깔끔하게 정리해줍니다.

```bash
snmp-check -t $target -c public
```

---

### **3. 주요 OID 목록**

| 정보 종류 | OID |
| :--- | :--- |
| 시스템 설명 | `1.3.6.1.2.1.1.1.0` |
| 시스템 가동 시간 | `1.3.6.1.2.1.1.3.0` |
| 시스템 호스트명 | `1.3.6.1.2.1.1.5.0` |
| 사용자 계정 | `1.3.6.1.4.1.77.1.2.25` |
| 실행 중인 프로세스 | `1.3.6.1.2.1.25.4.2.1.2` |
| TCP 로컬 포트 | `1.3.6.1.2.1.6.13.1.3` |


