20230106

오라클 클라우드에 프로젝트를 배포했는데 
원격 프로그램을 종료하면 프로젝트도 같이 종료되었다. 
알고보니 nohup이라는 명령어가 있었다.
장시간 실행이 필요한 서비스들을 백그라운드에서 실행해 준다고 한다.

nohup은 아래와 같이 사용할 수 있다.

```
nohup ./gradlew bootRun >Output.log 2>&1 &

nohup java -jar {프로그램 명} >Output.log 2>&1 &
```
이렇게 실행하면 output.log 라는 파일에 로그가 찍히고
백그라운드에서 프로그램이 실행된다.
로그 출력이 필요하지 않은 경우
```
nohup ./gradlew bootRun >/dev/null
```
로 지정하여 사용하면 된다.


