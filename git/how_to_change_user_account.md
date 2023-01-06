20230106

git bash에서 다른 유저로 로그인하는 방법

```
git config --global user.name = {username}
git config --global user.email = {useremail}
```
명령을 통해 깃허브 유저계정을 바꿀 수 있다.```

---
더욱 편하게 하는 방법을 찾다가 git alias라는 것에 대해 알게 되었다.
~/.gitconfig 를 vi로 열어 다음과같은 alias를 추가한다.

```
[alias]
	floppa = "!git config --global user.name {username} && git config --global user.email {user email} && git config user.email && echo changed to floppa dahee account"
```
원하는 명령어, 어떤 유저명인지, 어떤 이메일인지를 선택하고
선택된 계정 정보로 변경한 뒤 메시지를 출력해준다.
	
