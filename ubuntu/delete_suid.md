
230830

suid가 설정되어 있는지 확인하는 법
ls -l FILENAME

suid 가 적용되어있으면 권한에 x 대신에 s 로 적용 된 것을 확인할 수 있다.
chmod u-s FILENAME

적용 후 다시 ls -l 로 확인하면 s 가 x 로 변경되어 있다.
