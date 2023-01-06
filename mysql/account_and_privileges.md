20230106

오라클 클라우드에서 만든 mysql 데이터베이스에 접근하며 삽질을 했다.
1. inbound rule 적용하기
2. iptables로 포트 열기
```
# 3306 포트를 들어오는 ACCEPT 규칙을 추가
iptables -I INPUT -p tcp --dport 3306 -j ACCEPT 
# 3306 포트를 나가는 ACCEPT 규칙을 추가
iptables -I OUTPUT -p tcp --dport 3306 -j ACCEPT 
```
이후 iptables규칙을 조회하고 싶다면 (sudo)
```
iptables --list
iptables -L
```
3. telnet으로 외부에서 접속되는 지 확인하기
4. mysql root계정으로 접속하여 user id라는 계정을 만들고 스키마 접근 권한주기
```
$ mysql -u root -p password

use mysql;
create user 'user id'@localhost identified by 'user password';
create user 'user id'@'%' identified by 'user password';
grant all privileges on database name.* to 'user id'@'localhost' identified by 'user password';
flush privileges;

```
이제 외부에서 mysql 워크벤치를 통해 스키마에 접근할 수 있다.
오라클 클라우드를 사용하기 때문에 iptables를 사용했다.
firewalld를 사용하는 경우도 있으므로 어떤 방화벽을 사용하고있는지 체크한다.
root에 모든 데이터베이스 권한을 줘버리면 오라클클라우드에서 막아버린다.
이렇게되면 데이터베이스를 삭제하고 다시 깔아야 하니 주의한다.
