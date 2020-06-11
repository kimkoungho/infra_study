# RabbitMQ 설치하기 (centos) 

## Erlang 설치
 
```shell script
$ cd ~
$ wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
$ rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
$ yum install erlang
```

- 수동 설치 
특정 버전의 erlang 을 설치하고 싶다면 [github](https://github.com/rabbitmq/erlang-rpm/releases) 에서 다운받아 yum 으로 설치가능함  

## RabbitMQ 서버 설치    
RabbitMQ 클라우드를 이용한 설치 방법과 .rpm 파일을 다운받아 설치하는 방법이 있음

### 클라우드를 이용한 설치 
ㄴ 디펜던시 관리 때문에 이 방법을 관장하고 있음  
```shell script
# gpg key
# 2개의 키를 모두 가져와야 함  
# 2018 12 1 이후에 사용한 cloud key 
$ rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
# 2018 12 1 중단된 이전 cloud key 
$ rpm --import https://packagecloud.io/gpg.key

# yum 을 이용한 설치시 필요한 서명키 
$ rpm --import https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc 
```
/etc/yum.repos.d/ 아래에 .repo 파일을 추가해야함
 
```shell script
[bintray-rabbitmq-server]
name=bintray-rabbitmq-rpm
baseurl=https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.8.x/el/7/
gpgcheck=0
repo_gpgcheck=0
enabled=1
```

### .rpm 다운받아 설치 
[github](https://github.com/rabbitmq/rabbitmq-server/releases) 에서 특정 버전을 다운받아서 설치 진행 
ㄴ 사내망만 접근이 가능한 서버의 경우 수동 설치를 진행해야 할 수 있음  
  
수동 설치할 때 주의할 점은 erlang 버전과 매칭이 필요함  
- [rabbitmq 의 erlang 버전](https://www.rabbitmq.com/which-erlang.html) 정보를 참고해서 해당 erlang 을 다운받아야 함  
- 



## 래퍼런스 
- [공식 RabbitMQ 설치 페이지](https://www.rabbitmq.com/install-rpm.html#downloads)