---
layout: page
title: toast
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```js
function stackTrace() {
    var ThreadDef = Java.use('java.lang.Thread');
    var ThreadObj = ThreadDef.$new();



    console.warn("===================================[Stack Trace]===================================");
    var stack = ThreadObj.currentThread().getStackTrace();
    for (var i = 0; i < stack.length; i++) {
        console.log(i + " => " + stack[i].toString());
    } console.warn("===================================================================================");
}

function main() {
    console.log("Enter the Script!");
    Java.perform(function x() {
        console.log("script!!!!");
		
		var Toast = Java.use('android.widget.Toast');
		var makeText = Toast.makeText;
		var String = Java.use("java.lang.String");
		
		makeText.overload("android.content.Context","java.lang.CharSequence","int").implementation = function (context, content, time){
			console.log("toast!!!!!!!!!");
			stackTrace();
			//var content = "test!!!!".repeat(10);
			//var hookContent = String.$new(content);
			
			//return this.makeText(context, hookContent, time);
		}
		

    })
}


setImmediate(main);

```


