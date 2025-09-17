---
layout: page
title: phone_number_change
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---


```js
var Class_name = Java.use("android.telephony.TelephonyManager");

  Class_name.getLine1Number.overload().implementation = function() {
      var ret = this.getLine1Number();
      console.warn(ret);
      
      return "+821085051356";
  }
```

