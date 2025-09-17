

```js
var Class_name = Java.use("android.telephony.TelephonyManager");

  Class_name.getLine1Number.overload().implementation = function() {
      var ret = this.getLine1Number();
      console.warn(ret);
      
      return "+821085051356";
  }
```

