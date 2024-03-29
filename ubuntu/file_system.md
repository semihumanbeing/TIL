231016

## 파일 시스템 관리

/etc/fstab 파일 : 리눅스 시스템이 부팅될 때 자동으로 마운트할 파일 시스템의 목록을 가진 설정 파일

uuid: 파일 시스템을 구분하는 숫자 (32자리 16진수)
블록 디바이스 정보 보기: blkid, lsblk -f <- 를 활용하여 uuid 를 확인할 수 있다.

- 마운트, 언마운트시 내용이 /etc/mtab 파일에 기록됨
- 옵션 없이 mount 명령을 사용하면 현재 마운트 되어있는 파일 시스템을 보여준다.
- 기본 마운트 옵션: rw, suid, dev, exec, auto, nouser, async
  ㄴmount 명령에서 옵션 -o를 사용할 떄 적용할 수 있음

### Mount 명령
- 파일 시스템을 마운트하는 명령
ㄴ mount -a [options] [-t type] 
   :-a 는 /etc/fstab 파일에 기록된 모든 파일 시스템을 마운트한다. -t 가 추가되면 해당 유형의 파일 시스템만 마운트한다.
ㄴ mount [options] [-o mount_options] device | derectory 
   : -a 가 사용되지 않으면 지정된 장치에 해당하는 것을 찾아 마운트 한다. 
ㄴ mount [options] [-t type]] [-o mount_options] device directory
   : /etc/fstab 파일과 무관한 마운트 방법
   : 명령을 실행하기 전에 마운트 지점 (디렉터리)를 생성해야 한다.
   : ex) mount -t ext4 /dev/sdb1 /mnt/data1

### umount 명령
- 디렉터리에 마운트되어있는 저장장치를 해당 디렉터리로부터 분리한다.
ㄴ umount -a [-nv] [-t]
ㄴ umount [-nv] device | directory

## 파티션 관리
- 파티션: 물리적 저장장치를 논리적으로 분할한 고정 크기 구역
- 파티션은 자신만의 디바이스 파일을 가진다.

- 디스크를 분할하는 이유
ㄴ 멀티 부팅을 위해 여러 운영체제를 별도의 파티션에 설치함
ㄴ 특정 파티션이 손상되더라도 다른 파티션의 데이터는 보존된다
ㄴ 파티션 별로 다른 파일시스템을 만들수 있다.
ㄴ /boot 영역을 별도의 파티션으로 분리하여 빠르게 부팅한다.
ㄴ /var 영역을 별도의 파티션으로 만들어 /(루트) 영역의 가용 공간이 줄어드는 문제를 방지한다.
ㄴ 가상 메모리로 사용될 스왑 영역을 별도의 파티션으로 구성한다.

파티션 관리 도구 
1. fdisk (gpt는 지원하지 않음)
2. parted (mbr과 gpt를 지원)
3. gdisk (gpt를 지원)

- parted -l 또는 fdisk -l 명령은 모든 블록 디바이스에서 파티션 정보를 보여준다.
- mkpart primary 0% 8GB 이런식으로 파트를 나눌 수 있다.

- mkfs 명령: 파티션이나 논리 볼륨에 리눅스 파일 시스템을 만드는 명령
mkfs -t ext4 /dev/myVg/my_volumn 
- fsck 명령: 파일 시스템 무결성 검사 및 손상된 파일 고치기

### Swap 영역
- 특정 파티션이나 파일을 스왑 영역으로 지정하여 사용할 수 있음
- 메모리 사용량 확인하는 free 명령으로 스왑 메모리도 알 수 있다.
- mkswap [device] : 스왑 파티션을 만듬
- swapon [device] : 활성화 시키기
- 부팅할 때마다 사용하려면 /etc/fstab 파일에 기록한다.
- df 명령: 마운트 되어있는 파일 시스템의 공간 사용 정보를 보여준다.
ㄴ옵션:  -h (용량의 단위 kb) -T (파일시스템 유형을 출력) -i (inode의 사용량)
- du 명령: 디렉터리의 디스크 사용량을 표시하는 명령
ㄴ -a 는 파일의 디스크 사용량도 출력, -s 는 주어진 디렉터리 또는 파일만
ㄴ 단순 du 명령은 현재 디렉터리와 모든 서브 디렉터리의 사용량을 표시한다.







