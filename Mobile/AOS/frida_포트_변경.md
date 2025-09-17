---
layout: page
title: frida_포트_변경
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```bash title="adb shell"
./frida-server -l localhost:8088
```

```powershell title="local 포트포워딩"
adb forward tcp:8088 tcp:8088
```

```powershell title="frida script 실행"
frida -H localhost:8088 -f com.sampleapp -l test.js
```
