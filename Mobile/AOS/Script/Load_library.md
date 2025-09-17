

```js
var dlopen = new NativeFunction(Module.findExportByName(null, 'dlopen'), 'pointer', ['pointer', 'int']);

Interceptor.replace(dlopen, new NativeCallback(function(path, mode) {
    console.log("dlopen(" + "path=\"" + Memory.readUtf8String(path) + "\"" + ", mode=" + mode + ")");
    var name = Memory.readUtf8String(path);
    if (name !== null) {
        console.log("[*] dlopen " + name);
    }
    return dlopen(path, mode);
}, 'pointer', ['pointer', 'int']));


Java.perform(function() {
    const System = Java.use('java.lang.System');
    const Runtime = Java.use('java.lang.Runtime');
    const VMStack = Java.use('dalvik.system.VMStack');

    System.loadLibrary.implementation = function(library) {
        try {
			/*
			if(library == "dxbase"){
				console.log('System.loadLibrary("' + library + '")');
				Runtime.getRuntime().loadLibrary0(VMStack.getCallingClassLoader(), library);
				//Thread.sleep(10000);
			}
			else{
				console.log('System.loadLibrary("' + library + '")');
				//Thread.sleep(10000);
				Runtime.getRuntime().loadLibrary0(VMStack.getCallingClassLoader(), library);
			}
			*/
            console.log('System.loadLibrary("' + library + '")');
            Runtime.getRuntime().loadLibrary0(VMStack.getCallingClassLoader(), library);
        } catch(ex) {
            console.log(ex);
        }
    };
    
    System.load.implementation = function(library) {
        try {
            console.log('System.load("' + library + '")');
            Runtime.getRuntime().nativeLoad(library, VMStack.getCallingClassLoader());
        } catch(ex) {
            console.log(ex);
        }
    };
});
```


