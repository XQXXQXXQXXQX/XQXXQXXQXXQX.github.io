---
layout: page
title: pgp_파일_해독
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

개인키 파일(.asc 또는 .gpg)이  존재한다면 kali 로 옮겨서 크랙 진행

- gpg2john: John the Ripper(비밀번호 크래킹 도구)의 하위 유틸리티로, GPG 키 파일(.asc 또는 .gpg 형식)에서 패스프레이즈 해시를 추출합니다. GPG 키는 비밀 키를 보호하기 위해 패스프레이즈로 암호화되는데, 이 도구는 그 해시를 John the Ripper가 처리할 수 있는 형식으로 변환합니다.

```bash
gpg2john tryhackme.asc > hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```


크랙 후 Target 서버에서 gpg 명령 존재 확인 후 pgp 파일 해독 진행

```bash
# 기존 개인키 파일 임포트 시킴
gpg --import tryhackme.asc
```

![Pasted_image_20250904000459.png](/image/Pasted_image_20250904000459.png)

```bash
gpg -o decrypted_msg.txt -d credential.pgp
# john으로 크랙된 passphrase 입력 후 진행
```

![Pasted_image_20250904000536.png](/image/Pasted_image_20250904000536.png)

