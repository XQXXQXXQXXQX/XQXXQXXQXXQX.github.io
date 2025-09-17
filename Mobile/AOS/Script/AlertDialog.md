

```js title:"AlertDialog.js"
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
