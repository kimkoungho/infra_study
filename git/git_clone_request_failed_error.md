# centos 6 에서 git clone 시 에러가 발생함 

```shell script
$ git clone https://github.com/~
Initialized empty Git repository in ...
error:  while accessing https://github.com/~

fatal: HTTP request failed
```

- nss, curl 업데이트 
```shell script
$ sudo yum update -y nss curl
```

성공함 
