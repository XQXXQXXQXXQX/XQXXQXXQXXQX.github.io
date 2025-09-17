---
layout: page
title: frida_포트_변경_iOS
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

## ✅ iOS에서 Frida포트 변경 (launchd plist 수정)

### 1. 📂 plist 경로 확인

예를 들어:

```bash
/var/jb/Library/LaunchDaemons/re.frida.server.plist
```

이게 실제 plist 이름이라면 다음으로 진행하세요.

---

### 2. 📝 plist 편집

```bash
vi /var/jb/Library/LaunchDaemons/re.frida.server.plist

```

찾아야 할 부분은 `<key>ProgramArguments</key>` 아래입니다.

기존 내용

```xml
<key>ProgramArguments</key>
<array>
    <string>/var/jb/usr/sbin/frida-server</string>
</array>
```

이걸 아래처럼 **포트 추가**해서 수정 (예: `8878` 포트로):

```xml
<key>ProgramArguments</key>
<array>
    <string>/var/jb/usr/sbin/frida-server</string>
    <string>-l</string>
    <string>0.0.0.0:8878</string>
</array>
```

---

### 3. 🔁 변경 적용

plist를 저장한 후:

```bash
launchctl unload /var/jb/Library/LaunchDaemons/re.frida.server.plist
launchctl load /var/jb/Library/LaunchDaemons/re.frida.server.plist
```

## Windows

[https://github.com/libimobiledevice-win32/imobiledevice-net/releases](https://github.com/libimobiledevice-win32/imobiledevice-net/releases)

[libimobiledevice.1.2.1-r1122-win-x64.zip](attachment:2d8c12e9-a9c3-41fd-8aa6-19e6190b6484:libimobiledevice.1.2.1-r1122-win-x64.zip)

git에서 iproxy를 설치해서 포트포워딩 해준다. (환경변수 설정)

![image_(3).png](/image/image_(3).png)
