# Yum group install
yum 의 "groupinstall" 은 패키지를 하나씩 설치하지 않고도 패키지의 그룹을 쉽게 설치할 수 있음   
아래의 패키지들을 설치한다고 가정해보자   
```shell script
$ yum install gcc
$ yum install glibc
$ yum install make
```
이런 명령들을 한번에 할 수 있다면 좋을 것이라고 생각해서 패키지를 그룹화 한 것   

## 사용법 
- 그룹 리스트 출력 
```shell script
$ yum grouplist
```
ㄴ 해당 그룹들에서 설치하고자 하는 그룹을 선택해 설치 가능 

- 그룹의 정보 
```shell script
$ yum groupinfo [group]
```

- 특정 그룹 설치 
```shell script
$ yum groupinstall [group]
```

- 그룹 삭제
```shell script
$ yum groupremove [group]
```

- 그룹 업데이트
```shell script
$ yum groupupdate [group]
```

- 모든 개발 라이브러리 설치 
```shell script
$ yum groupinstall 'Development Tools'
```


## 래퍼런스 
- [제타위키](https://zetawiki.com/wiki/YUM_'Development_Tools'_%EA%B7%B8%EB%A3%B9_%EC%84%A4%EC%B9%98)
- [unixmen](https://unixmen.com/yum-groupinstall-a-quick-introduction/)
