

```js
var count = 1;

Interceptor.attach(Module.findExportByName(null, 'pthread_create'), {
	onEnter: function(args) {
		//console.log("------------------pthread_create() overloaded------------------");
		//console.log("[-] PTR : "+args[0]+" / "+args[1]+" / "+args[2]+" / "+args[3]+" / "+count+"\n");
		count++;
		
		var ptr2 = args[2];

			Interceptor.attach(ptr2, {
				onEnter: function(args) {
				 
					var xbase = String(DebugSymbol.fromAddress(this.context.pc));
					/*
					if (xbase.indexOf('libdxbase') != -1) {
					  console.log("[!] pthread args : " + xbase);
					  Thread.sleep(3000);
					}
					*/
					console.log("[!] pthread args : " + xbase);
				}
			});
	}
});
```


