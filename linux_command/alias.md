# 리눅스 명령어 alias
자주 사용하는 긴 명령어 조합을 간단하게 alias 로 등록하여 사용할 수 있음   

## alias 생성하기  
만드는 방법은 alias 별칭="명령어" 형식으로 생성  
```shell script
$ alias psa="ps aux" 
$ psa
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 138112  7508 ?        Ss   08:18   0:00 sudo /usr/sbin/sshd -D
```

alias 명령어를 이용한 방식은 추후에 지워질 수 있기 때문에 
일반적으로 환경변수 파일인 .bash_profile, .bash_rc 등에 등록해서 사용함  
  

## alias 삭제하기 
unalias 별칭
```shell script
$ unalias psa
$ psa
bash: psa: command not found 
``` 

## alias 들 출력
```shell script
$ alias
alias psa="ps aux"
```


## 래퍼런스 
- [빌노트 블로그](https://withcoding.com/121)