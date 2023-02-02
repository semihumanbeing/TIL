20230126
```
sudo apt-get update && sudo apt-get upgrade

sudo apt-get install openjdk-11-jdk

```
자바가 설치되었는지 확인하기
```
java -version
javac -version
```

JAVA_HOME 시스템 변수 설정하기

```
vim ~/.bashrc
```
bashrc를 열고 아래 내용을 추가한다.

```
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH=$PATH:$JAVA_HOME/bin
```
추가한 뒤 아래 명령어로 적용한다.
```
source ~/.bashrc 
```
설정되었는지 아래 명령어로 체크한다.
```
echo $JAVA_HOME
```


