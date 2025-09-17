

# 웹 콘텐츠 탐색 가이드

웹 서버를 공격할 때, 눈에 보이는 페이지 외에 숨겨진 디렉터리, 파일, 관리자 페이지 등을 찾는 것은 매우 중요합니다. 이 가이드는 웹 서버의 숨겨진 콘텐츠를 효율적으로 찾는 방법을 소개합니다.

---

### **1. 디렉터리 및 파일 탐색 (Brute-forcing)**

- **목표:** 단어 목록(Wordlist)을 기반으로 서버에 존재하는 디렉터리와 파일 경로를 추측하여 찾아냅니다.
- **핵심 도구:** `gobuster`, `feroxbuster`, `ffuf`

```bash title="Gobuster - 디렉터리 탐색"
# -u: 타겟 URL
# -w: 사용할 단어 목록 파일
# -t: 동시에 보낼 요청 수 (스레드)
# -o: 결과를 저장할 파일
gobuster dir -u http://$target -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -t 50 -o gb_dirs.txt

gobuster dir -u http://$target -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -o gb_dirs2.txt -b  403,404,400
```

```bash title="Gobuster - 특정 확장자 파일 탐색"
# -x: 탐색할 파일 확장자 목록
gobuster dir -u http://$target -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 50 -x php,xml,html,js,sql,gz,zip,txt,bak -r -o gb_files.txt

gobuster dir -u http://$target -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -t 50 -x php,xml,html,js,sql,gz,zip,txt,bak -r -o gb_files2.txt
```


```
Error: the server returns a status code that matches the provided options for non existing urls. http://localhost:8080/15cce213-c378-4367-806a-461973c15734 => 403 (Length: 865). To continue please exclude the status code or the length
```
이런 결과가 나올 경우 -b 403 옵션을 주면 됨

---

### **2. 가상 호스트 탐색 (Virtual Host Enumeration)**

- **목표:** 동일한 IP 주소에서 여러 개의 다른 웹사이트를 운영하는 경우(가상 호스트), 숨겨진 하위 도메인이나 다른 도메인 이름을 찾아냅니다.
- **핵심 전략:** HTTP 요청의 `Host` 헤더 값을 변경하며 서버의 반응을 확인합니다.

```bash title="ffuf - Host 헤더 퍼징"
# -H: 수정할 HTTP 헤더. FUZZ 키워드에 단어 목록의 값이 삽입됨
# -hc: 결과에서 제외할 HTTP 상태 코드 (Hide Code)
# subdomains.txt: 'admin', 'dev', 'test' 등 하위 도메인 이름 목록
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://asdf.thm/' -H "HOST: FUZZ.asdf.thm" -fw 522
```

```bash title="(do --hh 402 f.ex on the character count with wfuzz to filter it out)"
wfuzz -u http://$target/FUZZ --hc 404 -c -t 40 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

```bash
wfuzz -u http://$target/FUZZ --hc 404 -c -v -t 40 -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt
```

```bash
wfuzz -u http://$target/FUZZ --hc 404 -c -v -t 40 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```bash
wfuzz -u http://$target/FUZZ --hc 404 -c -v -t 40 -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
```

---

### **3. 자동화된 크롤링 (Automated Crawling)**

- **목표:** 웹사이트를 자동으로 돌아다니며 페이지에 링크된 모든 경로, 특히 JavaScript 파일 내에 숨겨진 API 엔드포인트나 경로를 수집합니다.
- **핵심 도구:** `katana`

```bash title="Katana - 웹사이트 크롤링"
# -u: 타겟 URL
# -jc: JavaScript 파일 내의 경로까지 분석
# -silent: 불필요한 출력 최소화
katana -u http://$target -jc -silent -o katana_endpoints.txt

katana -u http://$target$ -jc --depth 4 -silent
```

---

### **4. 추천 Wordlists (`/usr/share/seclists/` 기준)**

- **`Discovery/Web-Content/raft-large-directories.txt`**: 디렉터리 탐색 시 가장 먼저 사용해볼 만한 훌륭한 목록입니다.
- **`Discovery/Web-Content/raft-large-files.txt`**: 파일 탐색에 효과적인 포괄적인 목록입니다.
- **`Discovery/DNS/subdomains-top1million-5000.txt`**: 가상 호스트 및 하위 도메인 탐색에 매우 유용합니다.
- **`Discovery/Web-Content/common.txt`**: 작고 빠르지만, 일반적인 파일(예: `index.php`, `admin.php`)을 찾는 데 효과적입니다.
- **`Discovery/Web-Content/big.txt`**: 매우 광범위한 목록으로, 다른 목록으로 결과를 찾지 못했을 때 사용해볼 만합니다.


```txt
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt
/usr/share/seclists/Discovery/Web-Content/big.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-words.txt
```
