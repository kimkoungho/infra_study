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
  
```shell script
$ sudo yum install -y rabbitmq-server-3.7.7-1.el7.noarch.rpm
# rabbitmq 서버 시작 (데몬) 
$ sudo service rabbitmq-server start
# 확인 
$ sudo service rabbitmq-server status

```

## Management UI 플러그인 활성화
rabbitmq 는 Management UI 라는 관리도구를 제공하는데, 이 기능을 사용하기 위해서 플러그인을 활성화 해야 함  
```shell script
$ rabbitmq-plugins enable rabbitmq_management
```
- 추후에 http://host:15672 로 접속하면 WEB UI 에 확인가능하다  


## User 권한 
rabbitmq 가 최초에 기동되면 / 인 virtual host 와 guest/guest 계정이 생성됨  
ㄴ 일반적으로 보안 처리를 위해서 guest 계정은 삭제하고 새로운 게정을 만들어 사용  
guest 계정은 기본적으로 localhost 에서만 접속이 가능하고 remote 에서는 사용할 수 없는 계정  
  
- configure
리소스 (queue, exchange 등) 을 선언할 수 있는 권한  

- write 
exchange 에 대한 publish 권한  

- read
queue 에 대한 consume 권한  

GUI 에서 계정의 권한을 설정 가능한데, http://host:15672 로 접속하면 확인 가능함  
- Tags: 계정의 권한을 부여하는 부분인데 Admin, Monitoring, Policymaker, Management, None 이 있음   
Admin: 관리자 계정
Monitoring: 조회만 가능한 게정 
 

## User 추가 
rabbitmq 에 접속할 수 있는 관라자를 추가해준다    
```shell script
# 관리모드를 활성화 한다 
$ sudo rabbitmq-plugins enable rabbitmq_management

$ sudo rabbitmqctl add_user [user] [password]
$ sudo rabbitmqctl set_user_tags [user] administrator

# 추가된 관리자 확인 
$ sudo rabbitmqctl list_users
```
- 위에서 설명했듯이 관리자 계정은 1개는 꼭 필요함  
- guest 는 localhost 에만 접속이 가능하기 때문  

## rabbitmq 설정 파일 
[RabbitMQ 설정 가이드](https://www.rabbitmq.com/configure.html#supported-environment-variables) 를 따라서 설정파일을 작성하면 됨 
```shell script
$ sudo vi /etc/rabbitmq/rabbitmq.config
# 설정 내용을 기입

```
ㄴ 주의사항은 마지막 . 가 들어가야함 ...

## Virtual Hosts
RabbitMQ 는 connection, exchange, queue, binding, user, policy 들을 virtual hosts 를 통해서
논리적인 그룹으로 분리해서 운영할 수 있음  
vhost 단위로 자원에 대한 권한을 갖음  
사용자는 전체 권한을 가질수 없고 하나 이상의 vhost 단위로 권한을 부여받음  
 

## 클러스터 구성하기
### 노드 이름(식별자) 
각 노드는 노드 이름으로 식별됨  
예를들어 rabbit@node1.messagging.svc.local 의 경우 rabbit 이라는 접두어와 node1.messagging.svc.local 이라는 호스트를 가진 노드  
클러스터 상에서 노드 이름은 고유해야함  
ㄴ rabbit 이 하나의 클러스터 그룹   
ㄴ @ 뒤가 해당 노드의 식별자라고 보면됨  

### check 호스트 이름 
클러스터를 구성하면 호스트 이름이 노드의 이름으로 사용되기 때문에 각 호스트 이름으로 해당 노드에 접속할 수 있어야 함  
- DNS 레코드 : DNS 상에 해당 호스트 이름이 등록되어 있는 것 
- /etc/hosts : 클러스터의 각 노드가 대상 호스트를 인식할 수 있도록 ip 와 매핑되는 호스트 정보를 기입  

rabbitmq 는 2가지 호스트 등록방식을 지원하는데 FQDN(정규화 된 도메인 이름) 과 간략화된 호스트 임 
ex) node1.messagging.svc.local
- FQDN은 node1.messagging.svc.local 로써 호스트의 풀경로를 의미
- 간략화된 호스트는 node1 만 호스트로 지정하는 방식 (default)
ㄴ FQDN 방식을 사용하기 위해서는 별도의 추가 설정이 필요함
ㄴ [RabbitMQ 설정 가이드](https://www.rabbitmq.com/configure.html#supported-environment-variables)
  
개인적으로 /etc/hosts 파일을 수정하는 것은 나중에 서버의 증설, 이전 등에서 굉장히 불편하기 때문에 DNS 레코드 방식이나 FQDN 방식을 추천한다  

### 

### 실제 구성 
 


## 래퍼런스 
- [공식 RabbitMQ 설치 페이지](https://www.rabbitmq.com/install-rpm.html#downloads)
- [RabbitMQ 클러스터](https://www.rabbitmq.com/clustering.html)
- [RabbitMQ 설정 가이드](https://www.rabbitmq.com/configure.html#supported-environment-variables)
- [github 블로그](http://gjchoi.github.io/rabbit/rabbit-mq-%EA%B6%8C%ED%95%9C%EA%B4%80%EB%A6%AC/)