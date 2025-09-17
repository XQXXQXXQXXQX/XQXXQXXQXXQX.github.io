

```js title:"native.js"


function next_f(sub2)
{
    Interceptor.attach(sub2, {
		onEnter: function(args) {
			console.warn('[+] sub2 !!! : ' + sub2);
			
			//this.context.x0 = ptr("0x0");
			//console.error('x : ', this.context.x0);
			//console.log("#### : " + String(DebugSymbol.fromAddress(this.context.x2)));
			//console.log("args[0] : " + args[0]);
			//console.log("args[1] : " + args[1]);
			//console.log("args[2] : " + args[2]);
			//args[0] = ptr("0x2");
			//console.log("new args[0] : " + args[0]);
			//console.log("\n\x1b[31margs[0]:\x1b[0m \x1b[34m" + args[0].readUtf8String() + ", \x1b[32mType: ");
			//console.log('called from:\n' + Thread.backtrace(this.context, Backtracer.FUZZY).map(DebugSymbol.fromAddress).join('\n') + '\n');
			Thread.sleep(1);
		}, 
		onLeave: function (retval) { 
				
				console.error("[-] ret: " + retval);
				Thread.sleep(1);
				
				//var new_retval = ptr("0x0");
				//retval.replace(new_retval);
				//console.warn("[-] new ret: " + retval);
				

				

		}
	});
}


var so_name = null;
var BaseAddress = null;
var sub = null;
var sub2 = null;

Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
    onEnter: function (args) {
        var path = args[0].readUtf8String();
        //console.log("[*] android_dlopen_ext(\" " + path +" \")");
		
		if(path.indexOf('libdxbase.so') !== -1) {
			console.log("[*] android_dlopen_ext(\" " + path +" \")");
			this.hook = true;
		}
    },
	onLeave: function(retval) {
		if(this.hook) {
			var nativefunc = 'kill';
			so_name = 'libdxbase.so';
			BaseAddress = Module.findBaseAddress('libdxbase.so');
			sub = BaseAddress.add(0x81c8);
			sub2 = BaseAddress.add(0x19d20);
			
			
			console.log('[*] BaseAddress : ' + BaseAddress);
			console.log('[*] SubAddress : ' + sub);
			
			if(BaseAddress !== null) {
				
				Interceptor.attach(sub, {
					onEnter: function(args) {
						console.warn('[+] !!!!!!!!!!!!!!!!!');
						
						//this.context.x1 = ptr("0x0");
						//this.context.x2 = ptr("0x0");
						console.error('x0 : ', this.context.x0);
						//console.error('x2 : ', this.context.x2);
						//console.log("#### : " + String(DebugSymbol.fromAddress(this.context.x2)));
						
						
						//console.log("args[0] : " + args[0]);
						//console.log("args[1] : " + args[1]);
						//console.log("args[2] : " + args[2]);
						//args[0] = ptr("0x2");
						//console.log("new args[0] : " + args[0]);
						//console.log("\n\x1b[31margs[0]:\x1b[0m \x1b[34m" + args[0].readUtf8String() + ", \x1b[32mType: ");
						//console.log('called from:\n' + Thread.backtrace(this.context, Backtracer.FUZZY).map(DebugSymbol.fromAddress).join('\n') + '\n');
						Thread.sleep(1);
						
					}, 
					onLeave: function (retval) { 
							
							console.error("[-] ret: " + retval);
							Thread.sleep(1);
							
							var new_retval = ptr("0x0");
							retval.replace(new_retval);
							console.warn("[-] new ret: " + retval);
							
							
							// sub2
							next_f(sub2);
							
	
					}
				});
				
				/*
				Interceptor.attach(Module.findExportByName(so_name, nativefunc), {
					onEnter: function(args) {
						//var check_file = args[0].readUtf8String();
						
						console.warn('[*] ' + nativefunc + ' called !');
						//console.log("\n\x1b[31margs[0]:\x1b[0m \x1b[34m" + args[0].readUtf8String() + ", \x1b[32mType: ");
						//console.log('called from:\n' + Thread.backtrace(this.context, Backtracer.FUZZY).map(DebugSymbol.fromAddress).join('\n') + '\n');
						
						//if(check_file.indexOf('frida') !== -1) {
						//	this.hook2 = true;
						//}
					}, 
					onLeave: function (retval) { 
							
							//if(this.hook2){
								//var new_retval = ptr("0x0");
								//var new_retval = NULL;
								//retval.replace(new_retval);
							//}
							
							//Thread.sleep(30);
							console.warn("[-] ret: " + retval.toString() ); // after call 

					}
				});
				*/

			}
		}
	}
});
```


