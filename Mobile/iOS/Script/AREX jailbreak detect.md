
```js title="arex.js"
if(ObjC.available){
    try{
        var module_base = Module.findBaseAddress('AREX-RV');  //앱 Base명 입력
        var sub = module_base.add(0x152284);   // 오프셋 값 입력

        console.warn(module_base);
 
         Interceptor.attach(sub, {   
            onEnter: function (args) {
				console.log("");
                console.log("onEnter!!!!!!!");
				
                //console.warn('\t[+] Backtrace:\n\t' + Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n\t'));
                
               
                console.log("[*] x0  : " + JSON.stringify(this.context.x0));
                this.context.x0 = 0x1;
                // console.warn("[*] change x0  : " + JSON.stringify(this.context.x0));


				
                // console.warn("args: " + args[1]+", "+args[2]+", "+args[3]);
            },
            onLeave: function (retval) {
                // console.warn("[*] change x0  : " + JSON.stringify(this.context.x0));
				// console.log("\t[-] Type of return value: " + typeof retval);
                // console.log("\t[-] Original Return Value: " + retval);
                // retval.replace(0x0); // 리턴값 변환
                // console.log("\t[-] Type of return value: " + typeof retval);
                // console.log("\t[-] Return Value: " + retval);
            
            }
        });

    }
    catch(err){
        console.log("[!] Error: " + err.message);
    }
}
else {
    console.log("Objective-C Runtime is not available!");
}
```