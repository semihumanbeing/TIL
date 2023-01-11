20230111

우분투 내에서 캐시 메모리 확인하기
```
# free -m
```
이러면 아래와 같이 정보를 보여준다.
(오라클 클라우드의 1기가짜리 램 pc ㅋㅋ)
```
              total        used        free      shared  buff/cache   available
Mem:            964         390         308           0         265         424
Swap:          4095         181        3914
```

아래의 명령어를 날리면 캐시를 삭제한다.
```
# sync && echo 3 > /proc/sys/vm/drop_caches
```

명령한 뒤 free -m 명령어를 쳐보면 공간이 늘어나는것을 확인할 수 있다.

crontab -e 명령을 한 뒤
들어가지는 에디터에서 제일 아랫줄에
```
0 4 * * * sync && echo 3 > /proc/sys/vm/drop_caches
```
를 입력하면 매일 새벽 4시에 캐시 메모리를 비워준다.
0 * * * 는 크론 시간으로 
https://crontab.guru/
이와 같은 사이트에서 원하는대로 설정하여 볼 수 있다.

