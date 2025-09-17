---
layout: page
title: capture_iOS
description: >
  This chapter covers the basics of content creation with Hydejack.
hide_description: true
sitemap: false
---

```js title="capture.js"
if (ObjC.available) {
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

} else {
    console.log("[-] ObjC 런타임을 찾을 수 없습니다.");
}

```

