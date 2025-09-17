

```js title:"stackTrace.js"
function stackTrace() {
    var ThreadDef = Java.use('java.lang.Thread');
    var ThreadObj = ThreadDef.$new();

    console.warn("===================================[Stack Trace]===================================");
    var stack = ThreadObj.currentThread().getStackTrace();
    for (var i = 0; i < stack.length; i++) {
        console.log(i + " => " + stack[i].toString());
    } console.warn("===================================================================================");
}
```


