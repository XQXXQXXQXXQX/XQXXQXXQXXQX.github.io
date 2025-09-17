---
layout: page
title: Integrity_Test
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```text
const-string v0, "Integrity Test"
const/4 v1, 0x0
invoke-static {p0, v0, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;
move-result-object v0
invoke-virtual {v0},Landroid/widget/Toast;->show()V
```


