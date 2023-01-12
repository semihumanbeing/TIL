20230112

열려있는 모든 포트 표시하기
```
netstat -nap
```

LISTEN 중인 포트 표시
```
netstat -nap | grep LISTEN
```

포트번호 상태확인
```
netstat -nap | grep 포트번호
```

방화벽 설정 정보 확인
```
iptables -nL
```

방화벽 외부 접속 허용하기
```
iptables -I INPUT 1 -p tcp --dport 12345 -j ACCEPT
```

방화벽 허용 포트 삭제
```
iptables -D INPUT -p tcp --dport 12345 -j ACCEPT
```

변경사항 저장하기
```
service iptables save
/etc/init.d/iptables restart
```

방화벽 켜기, 끄기
```
/etc/init.d/iptables start
/etc/init.d/iptables stop
```



