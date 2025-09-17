
```js title="AppIron.js"
if(ObjC.available){

    console.log("[*] 시작: 캡처 방지 우회 적용");

    // secureTextEntry 무효화 (항상 false로)
    if (ObjC.classes.UITextField && ObjC.classes.UITextField["- setSecureTextEntry:"]) {
        Interceptor.attach(ObjC.classes.UITextField["- setSecureTextEntry:"].implementation, {
            onEnter(args) {
                if (args[2].toInt32() === 1) {
                    console.log("[🔓] secureTextEntry → false로 변경 (캡처 가능하게)");
                    args[2] = ptr(0); // 강제 false
                }
            }
        });
    }

    // CALayer.setFilters 무효화 (빈 배열 전달)
    if (ObjC.classes.CALayer && ObjC.classes.CALayer["- setFilters:"]) {
        Interceptor.attach(ObjC.classes.CALayer["- setFilters:"].implementation, {
            onEnter(args) {
                const original = new ObjC.Object(args[2]);
                console.log(`[🧼] setFilters 감지됨 → 무효화 (기존 필터: ${original})`);

                // 빈 NSArray 전달
                const NSArray = ObjC.classes.NSArray;
                args[2] = NSArray.array(); // 빈 배열 전달
            }
        });
    }
    



    
    function next_f(sub2){
        Interceptor.attach(sub2, {   
            onEnter: function (args) {
                console.log("");
                console.error(sub2 + " onEnter!!!!!!!");
                
                console.warn('\t[+] Backtrace:\n\t' + Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n\t'));
                
               
                console.log("[*] x8  : " + JSON.stringify(this.context.x8));
                console.log("[*] x10  : " + JSON.stringify(this.context.x10));
                console.log("[*] x9  : " + JSON.stringify(this.context.x9));
                console.log("[*] x11  : " + JSON.stringify(this.context.x11));
                console.log("[*] x12  : " + JSON.stringify(this.context.x12));
                console.log("[*] x14  : " + JSON.stringify(this.context.x14));
                console.log("[*] x13  : " + JSON.stringify(this.context.x13));
                console.log("[*] x15  : " + JSON.stringify(this.context.x15));
                //this.context.x0 = 0x5;
                this.context.x8 = ptr("0x0");
                this.context.x10 = ptr("0x0");
                this.context.x9 = ptr("0x0");
                this.context.x11 = ptr("0x0");
                this.context.x12 = ptr("0x0");
                this.context.x14 = ptr("0x0");
                this.context.x13 = ptr("0x0");
                this.context.x15 = ptr("0x0");

                //console.warn("[*] change x13  : " + JSON.stringify(this.context.x13));
                // console.warn("[*] change x15  : " + JSON.stringify(this.context.x15));
                
                // console.warn("args: " + args[1]+", "+args[2]+", "+args[3]);
            },
            onLeave: function (retval) {
                console.error("sub2 onLeave!!!!!!!");
                // console.warn("[*] change x0  : " + JSON.stringify(this.context.x0));
                // console.log("\t[-] Type of return value: " + typeof retval);
                // console.log("\t[-] Original Return Value: " + retval);
                // retval.replace(0x0); // 리턴값 변환
                // console.log("\t[-] Type of return value: " + typeof retval);
                // console.log("\t[-] Return Value: " + retval);
            
            }
        });
    }

    function check_flow(sub2, n){
        Interceptor.attach(sub2, {   
            onEnter: function (args) {
                console.log("");
                console.error(n + " flow onEnter!!!!!!!");
                
                console.warn('\t[+] Backtrace:\n\t' + Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n\t'));
                
               
                console.log("[*] x0  : " + JSON.stringify(this.context.x0));
                // console.log("[*] x15  : " + JSON.stringify(this.context.x15));
                this.context.x0 = 0x0;
                // this.context.x13 = ptr("0x1");
                // this.context.x15 = ptr("0x1");

                // console.warn("[*] change x13  : " + JSON.stringify(this.context.x13));
                // console.warn("[*] change x15  : " + JSON.stringify(this.context.x15));
                
                // console.warn("args: " + args[1]+", "+args[2]+", "+args[3]);
            },
            onLeave: function (retval) {
                console.error("flow onLeave!!!!!!!");
                // console.warn("[*] change x0  : " + JSON.stringify(this.context.x0));
                // console.log("\t[-] Type of return value: " + typeof retval);
                // console.log("\t[-] Original Return Value: " + retval);
                // retval.replace(0x0); // 리턴값 변환
                // console.log("\t[-] Type of return value: " + typeof retval);
                // console.log("\t[-] Return Value: " + retval);
            
            }
        });
    }



    try{
        var module_base = Module.findBaseAddress('BusTago');  //앱 Base명 입력
        var sub = module_base.add(0x18CCB0);   // 오프셋 값 입력
        var sub2 = module_base.add(0x18CCD0);   // 오프셋 값 입력
        var sub3 = module_base.add(0x4944);   // 오프셋 값 입력
        var sub4 = module_base.add(0x4988);   // 오프셋 값 입력

        console.warn(module_base);

        // Native function connect
        var flag_connect=0;
        // Interceptor.attach(Module.findExportByName(null, 'connect'), {
        // onEnter: function (args) {
        //     var str = Memory.readUShort(args[1].add(2));
        //     var port = ((str & 0xFF) << 8) | ((str & 0xFF00) >> 8); 
        //     flag_connect=0;
        //     if (port == 27042) {
        //         //console.log("[N] Bypassed connect frida: ");
        //         flag_connect=1;
        //     }
        // },
        // onLeave: function (retval) {
        //     if (flag_connect) {
        //         retval.replace(-1);
        //         console.log("[N] Bypassed connect frida: ");
        //     }
        // }
        // });

        // check_flow(sub4,2);
        check_flow(sub2,1);
        // next_f(sub2);
        

         Interceptor.attach(sub, {   
            onEnter: function (args) {
				console.log("");
                console.error("[+] onEnter!!!!!!!");
				
                console.warn('\t[+] Backtrace:\n\t' + Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n\t'));
                
                

                console.log("[*] x0  : " + JSON.stringify(this.context.x0));
                console.log("[*] x19  : " + JSON.stringify(this.context.x19));
                console.log("[*] x2  : " + JSON.stringify(this.context.x2));
                console.log("[*] x22  : " + JSON.stringify(this.context.x22));
                console.log("[*] x4  : " + JSON.stringify(this.context.x4));
                console.log("[*] x25  : " + JSON.stringify(this.context.x25));
                // this.context.x25 = ptr("0x0");
                // this.context.x0 = ptr("0x0");
                // this.context.x19 = ptr("0x0");
                // console.warn("[*] change x0  : " + JSON.stringify(this.context.x0));

                // console.warn("args: " + args[0]+", "+args[1]+", "+args[2] + args[3]);
                
                
                
            },
            onLeave: function (retval) {
                // Thread.sleep(1);

                 //console.warn("[*] change x
                 // 0  : " + JSON.stringify(this.context.x0));
				 console.log("\t[-] Type of return value: " + typeof retval);
                 console.log("\t[-] Original Return Value: " + retval);
                // retval.replace(0x0); // 리턴값 변환
                //Thread.sleep(30);
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

