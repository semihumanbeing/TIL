20240208

dockerfile 을 만들고 난 뒤

docker build --tag imagename:version 
으로 태그를 지정하여 이미지를 생성할 수 있다.

docker images 로 생성된 이미지 확인
docker rmi 로 이미지 삭제가능 

파일을 tar 압축파일로 만드는법
docker save -o 압축파일명.tar 이미지이름
