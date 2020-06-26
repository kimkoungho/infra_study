# linux missing operand
shell script 를 실행하다 해당 오류가 발생하면 명령어를 실행하는 변수 중 하나가 설정되지 않았을 경우가 있음     
```shell script
$ readlink $FILE
readlink: missing operand
Try 'readlink --help' for more information. 
```
ㄴ 필자의 경우 readlink 에 사용하는 변수의 이름을 수정하다가 해당 에러가 발생함  
readlink 의 argument 가 제대로 설정되지 않아서 발생하는 에러  
 

[bash 명령어 및 인수 가이드](http://mywiki.wooledge.org/BashGuide/CommandsAndArguments)
[bash 가이드 매개변수](http://mywiki.wooledge.org/BashGuide/Parameters)



## 래퍼런스
- [stackExchange](https://askubuntu.com/questions/533217/changing-ownership-of-folder-not-working)