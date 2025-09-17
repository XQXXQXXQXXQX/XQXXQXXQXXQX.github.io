---
layout: page
title: AlertDialog
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```js
Java.perform(function () {
	// var ThreadDef = Java.use('java.lang.Thread');
	// var ThreadObj = ThreadDef.$new();
	var ClassName = Java.use('android.app.AlertDialog'); 
	ClassName.show.implementation = function () { 
		console.warn("\n[*] AlertDialog.show() called!\n"); 
		stackTrace();
	};
});
```
