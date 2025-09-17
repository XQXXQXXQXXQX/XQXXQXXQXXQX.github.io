---
layout: page
title: frida_포트_변경
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```bash
./frida-server -l localhost:8088
```

```powershell
adb forward tcp:8088 tcp:8088
```

```powershell
frida -H localhost:8088 -f com.sampleapp -l test.js
```
