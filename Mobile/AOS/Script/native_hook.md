---
layout: page
title: native_hook
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```js

Java.perform(function() {
	
	const System = Java.use('java.lang.System');
    const Runtime = Java.use('java.lang.Runtime');
    const VMStack = Java.use('dalvik.system.VMStack');

System.loadLibrary.implementation = function(library) {
		//console.log('[*] System.loadLibrary("' + library + '")');
		const loaded = Runtime.getRuntime().loadLibrary0(VMStack.getCallingClassLoader(), library);
		

			if(Module.findBaseAddress('libdxbase.so') != null) {
				var BaseAddress = Module.findBaseAddress('libdxbase.so');
				console.log('[*] libdxbase.so BaseAddress : ' + BaseAddress);
				
				var hook = BaseAddress.add(0x97b4);
				
				Interceptor.attach(hook, {
					onEnter: function(args) {
						console.warn('[*] !!!!!!');
						console.log('called from:\n' + Thread.backtrace(this.context, Backtracer.FUZZY).map(DebugSymbol.fromAddress).join('\n') + '\n');
						//console.error('0xdf14', args[0].readUtf8String());
						//Thread.sleep(3000);
						
						
					},
					onLeave: function(retval) {
						console.warn('[*] ???????');
						//Thread.sleep(3);
					}
				});
				
			}
		
		return loaded;
	}
	
	});
```


