20230111

오라클 클라우드 서버에서 돌아가던 프로그램이 갑자기 빌드가 안되었다.
에러 로그를 찾아보니 캐시 문제라고 한다.
```
gradle build --refresh-dependencies
gradle clean
```
그레이들에서 비울수 있는것은 다 비워보려 하는데
위의 두개 명령어도 실행이 느릴 정도였다.

Gradle의 dependencies 라이브러리 내의 캐시 폴더도 삭제했다.
```
rm -rf $HOME/.gradle/caches/
```

그레이들 실행을 하는데 3 busy Daemons could not be reused 로그가 나와서

```
gradle --stop 
```
으로 daemon 스레드도 멈췄다.
