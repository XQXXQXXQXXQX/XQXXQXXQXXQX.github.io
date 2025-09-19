
```powershell title="Set-ADAccountPassword"
# 변경할 새 비밀번호를 SecureString으로 변환
$newPassword = ConvertTo-SecureString 'NewPassword123!' -AsPlainText -Force

# Set-ADAccountPassword 사용
Set-ADAccountPassword -Identity SHAWNA_BRAY -NewPassword $newPassword -Reset
```

```powershell
Set-ADAccountPassword -Identity Darla_Winters -Reset -NewPassword (ConvertTo-SecureString 'Password00' -AsPlainText -Force)
```




