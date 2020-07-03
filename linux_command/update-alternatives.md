## update-alternatives 
jdk 여러개 사용하기 
```shell script
$ sudo update-alternatives --install <link> <name> <path> <priority>
# <link> : 실행파일 이름으로 /etc/alternatives/<name> 을 가리킨다
# <name> : 해당 링크 그룹의 대표 이름으로, 여러 가지 버전의 패키지들을 대표하는 이름으로 보면 될 것 같다
# <path> : alternatives 로 실제 연결할 실행파일 이름으로, 시스템에 설치한 패키지의 실행파일 이름이다
# <priority> : automatic 모드에서 어떤 것을 자동으로 선택해서 사용할지 결정할 때 사용되는 우선순위로, 높은 수가 더 높은 우선순위이다


```


https://www.whatwant.com/entry/update-alternatives-여러-버전의-패키지-관리하기 
