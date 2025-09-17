
python 코드 실행 시 ssl 에러가 나는 경우가 있음
![[image.avif]]

이런 경우 request.post 또는 request.get 함수에 `verify=False` 를 추가해주면 됨.

```python
print(requests.post(url, headers=headers, data=data, verify=False).text)
```

