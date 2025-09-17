
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=krb5asrep hashes.txt
```


#### NTLM

```bash title="원본 Hash 형식 (hashes.txt)"
Administrator:500:aad3b435b51404eeaad3b435b51404ee:549a1bcb88e35dc18c7a0b0168631411:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Lab:1000:aad3b435b51404eeaad3b435b51404ee:30e87bf999828446a1c1209ddde4c450:::
```

```bash
cut -d ':' -f 1,4 hashes.txt > ntlm_hashes.txt
```

```bash title="ntlm_hashes.txt"
Administrator:549a1bcb88e35dc18c7a0b0168631411
Guest:31d6cfe0d16ae931b73c59d7e0c089c0
Lab:30e87bf999828446a1c1209ddde4c450
```

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT ntlm_hashes.txt
```



