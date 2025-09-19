---
layout: page
title: Names_to_username_AD
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# AD 사용자 이름 생성 가이드

초기 정찰 단계에서 "John Smith"와 같은 직원들의 전체 이름 목록을 획득하는 경우가 많습니다. 이 가이드는 이러한 이름 목록을 실제 Active Directory 환경에서 사용될 가능성이 높은 사용자 이름 형식(예: `jsmith`, `john.s`, `smith.j` 등)으로 변환하고, 유효성을 검증하는 전체 워크플로우를 설명합니다.

- **[도구 GitHub 링크](https://github.com/mohinparamasivam/AD-Username-Generator)**

---

### **1. 왜 필요한가? (The Strategy)**

대부분의 조직은 일관된 사용자 이름 명명 규칙(Naming Convention)을 가집니다. 만약 한두 명의 유효한 사용자 이름(예: `j.smith`)을 안다면, 그 규칙을 유추하여 전체 직원 이름 목록에 적용할 수 있습니다. 이렇게 생성된 사용자 이름 목록은 무작위적인 추측보다 훨씬 더 정확하며, `kerbrute` 같은 도구와 함께 사용될 때 매우 효과적입니다.

---

### **2. 도구 설치 및 사용법**

```bash
git clone https://github.com/mohinparamasivam/AD-Username-Generator
cd AD-Username-Generator
```

```bash
# -u: "Firstname Lastname" 형식의 이름 목록 파일
# -o: 생성된 사용자 이름이 저장될 출력 파일
python3 username-generate.py -u full_names.txt -o potential_usernames.txt
```

- **입력 파일 예시 (`full_names.txt`):**
  ```
  John Smith
  Jane Doe
  Peter Jones
  ```

- **출력 결과물 예시 (`potential_usernames.txt`):**
  ```
  jsmith
  j.smith
  smithj
  johns
  ...
  jdoe
  j.doe
  ...
  ```

---

### **3. 전체 공격 워크플로우**

1.  **이름 수집 (Gather Names):** OSINT(공개 출처 정보 수집) 단계에서 회사 웹사이트, LinkedIn 등을 통해 직원 이름 목록을 수집합니다. 또는 시스템 침투 후 내부 문서에서 이름 목록을 확보합니다.

2.  **사용자 이름 생성 (Generate Usernames):** 위에서 설명한 `AD-Username-Generator`를 사용하여 수집된 이름 목록으로부터 가능한 모든 형식의 사용자 이름 목록을 생성합니다.

3.  **유효성 검사 (Validate with Kerbrute):** 생성된 사용자 이름 목록을 `kerbrute`의 입력값으로 사용하여, Active Directory에 실제 존재하는 유효한 계정들만 선별합니다.
```bash
kerbrute userenum --dc <DC_IP> -d <domain.local> potential_usernames.txt -o valid_users.txt
```

4.  **후속 공격 (Follow-up Attack):** 최종적으로 확인된 유효 사용자 목록(`valid_users.txt`)을 대상으로 AS-REP Roasting, Password Spraying 등 다음 단계의 공격을 수행합니다.

