

이 스크립트는 일반적으로 Windows 구성 문제 또는 잘못된 사례로 인해 발생하는 **LPE(Local Privilege Escalation**) 취약성을 식별하는 것을 목표로 합니다. 또한 일부 악용 및 악용 후 작업에 유용한 정보를 수집할 수 있습니다.

https://github.com/itm4n/PrivescCheck

---

```powershell
powershell -ep bypass
```


```powershell title="Basic checks only"
. .\PrivescCheck.ps1; Invoke-PrivescCheck
```


```powershell title="Extended checks + human-readable reports"
. .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended -Report PrivescCheck_$($env:COMPUTERNAME) -Format TXT,HTML
```


```powershell title="All checks + all reports"
. .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended -Audit -Report PrivescCheck_$($env:COMPUTERNAME) -Format TXT,HTML,CSV,XML
```

