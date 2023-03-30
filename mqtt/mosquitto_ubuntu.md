20230330

Ubuntu 서버에 Mosquitto 설치하고 사용하기
- 그런데 웹소켓을 사용함

최신 버전을 쓴다면 그냥 apt-get install mosquitto 하면 되지만...
1.6.9버전을 사용해야 하기 때문에 압축파일을 받아서 설정해야한다.

1. Mosquitto 1.6.9 버전 tar.gz파일을 다운로드, 압축풀기
https://launchpad.net/ubuntu/+source/mosquitto/1.6.9-1

```
wget https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/mosquitto/1.6.9-1/mosquitto_1.6.9.orig.tar.gz
```
```
tar -xzvf mosquitto_1.6.9.orig.tar.gz
```

2. Websocket을 사용하기 위한 환경 설정

config.mk 파일의 웹소켓 사용 설정
```
$ sudo vi config.mk

WITH_WEBSOCKETS:=yes 
```
mosquitto.conf 파일 제일 아래에 아래 내용을 추가한다.
```
listener 9001
protocol websockets

listener 1883
protocol mqtt
```


3. mosquitto 파일을 빌드한다.
```
$ cd ~/{소스파일 경로}/mosquitto-1.6.9/
$ make
$ sudo make install
```

4. mosquitto 파일이 있는 경로에서 아래 명령을 실행한다.
```
mosquitto -c mosquitto.conf
```

5. 브로커를 켜고 명령창으로 들어가지지 않기 때문에 nohup으로 스크립트를 만들었다.
vi nohup.sh
```
#!/bin/sh
nohup mosquitto -c mosquitto.conf >Output.log 2>&1 &
cat Output.log
```
스크립트를 mosquitto.conf가 위치한 곳에 두고 실행한다.
chmod 설정 하고 root에서 실행한다.



