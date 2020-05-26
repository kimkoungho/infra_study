# 아커스 설치 및 사용법 

## 아커스 설치

## 아커스 어드민 스크립트
[아커스 어드민 스크립트](https://github.com/naver/arcus/blob/master/docs/arcus-admin-script-usage.md)  
script/arcus.sh 로 아커스 클라우드를 관리하기 위해서 사용  

```shell script
Usage: ./arcus.sh -h
       ./arcus.sh [-z <zklist>] deploy <conf_file> | ping <service_code>
       ./arcus.sh [-z <zklist>] zookeeper init
       ./arcus.sh [-z <zklist>] zookeeper start|stop|stat
       ./arcus.sh [-z <zklist>] memcached register <conf_file> | unregister <service_code>
       ./arcus.sh [-z <zklist>] memcached start|stop|list <service_code>
       ./arcus.sh [-z <zklist>] memcached listall
       ./arcus.sh [-z <zklist>] quicksetup <conf_file>
       
  -z, --zklist    zookeeper ensemble ip:port list, The default is "localhost:2181"
                  example) -z 10.0.0.1:2181,10.0.0.2:2181,10.0.0.3:2181
  -h, --help      This small usage guide
  <conf_file>     Arcus cache cloud configuration file, having cache ip:port list and other settings.
                  Refer to scripts/conf/test.json file
  <service_code>  Arcus cache cloud name that identify each cache cloud uniquely.
                  We also call it "service code" as it's a kind of cache cloud service.
```
- 공통 인자인 zklist 는 zookeeper ensemble list 이며 기본 값은 localhost:2181 
- zklist 인자를 생략한다면 기본 값인 localhost:2181 이 적용됨 
ㄴ script/arcus.sh 파일 내부에 정의되어 있음 


```shell script
./arcus.sh [-z <zklist>] deploy <conf_file>
```
- 아커스 패키지를 zookeeper 장비와 memcached 장비에 설치 
ㄴ 해당 장비들에 arcus 패키지가 있어야 함 

```shell script
./arcus.sh [-z <zklist>] ping <service_code>
``` 
- 아커스 zookeeper 와 memcached 장비에 ping 을 수행
ㄴ memcached 관련정보는 <service_code> 를 이용해서 zookeeper 에서 조회함


```shell script
./arcus.sh [-z <zklist>] zookeeper init|start|stop|stat

```
- init: 설정 템플릿(conf/zoo.cfg) 를 이용해 zookeeper 설정을 생성한 후, 각 zookeeper 장비에 배포
ㄴ 아커스 클라우드를 위한 기본 디렉토리 생성  
- start: zookeeper 의 ensemble 의 모든 zookeeper process 구동 
- stop: zookeeper 의 ensemble 의 모든 zookeeper process 중지 
- stat: zookeeper 의 ensemble 의 모든 zookeeper process 상태를 조회

```shell script
./arcus.sh [-z <zklist>] memcached register <conf_file> 
./arcus.sh [-z <zklist>] memcached unregister <service_code>
```
- register: conf_file 의 아커스 캐시 정보를 읽어 zookeeper 에 등록 
ㄴ zookeeper 에 이미 service_code 가 존재한다면, conf_file 기준으로 새로운 service_code 생성 
- unregister: service_code 에 해당 하는 캐시 정보를 zookeeper 에서 삭제 

```shell script
./arcus.sh [-z <zklist>] memcached start|stop|list <service_code>
```
- start: service_code 에 해당하는 memcached process 를 구동 
- stop: service_code 에 해당하는 memcached process 를 정지 
- list: service_code 에 해당하는 memcached process list 조회 

```shell script
./arcus.sh [-z <zklist>] memcached listall
```
- 현재 zookeeper 에 등록된 모든 캐시들에 memcached 를 조회  

```shell script
./arcus.sh [-z <zklist>] quicksetup <conf_file>
```
- deploy, zookeeper init & start, memcached register & start 작업을 한번에 수행 

## 실제로 설치 해보기 
아커스 설치시 아커스 프로젝트를 배포하는 역할의 서버를 구성할 수 있음 
ㄴ 그냥 zookeeper 한대를 지정해도 상관없음   
ㄴ arcus 파일을 배포하는 용도임   
  
설치 순서   
1. arcus 구동하기 위한 디펜던시들 설치 
2. arcus git checkout
3. 배포를 위한 ssh key 설정 
4. 설정 파일 생성
5. 아커스 패키지 배포
6. 주키퍼 설치 
7. memcached 설치 

### arcus 구동하기 위한 디펜던시들 설치
[아커스 디펜던시 가이드](https://github.com/naver/arcus/blob/master/docs/howto-install-dependencies.md) 에 따라서 아래의 패키지들을 설치  

ex) centos
- jdk 와 ant 설치 
```shell script
# openjdk
$ sudo yum install -y java-1.8.0-openjdk
$ sudo echo -e "export JAVA_HOME=/etc/alternatives/java\nexport PATH=\$JAVA_HOME/bin:\$PATH" >> /etc/profile.d/java.sh
$ souce /etc/profile.d/java.sh

# ant
# /usr/local/src 에 설치함  
$ cd /usr/local/src
$ sudo wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.3-bin.tar.gz
$ sudo tar xvf apache-ant-1.9.3-bin.tar.gz
$ sudo mv /usr/local/src/apache-ant-1.10.7 /usr/local
$ cd /usr/local
$ sudo ln -s apache-ant-1.10.7 ant
$ sudo echo -e "export ANT_HOME=/usr/local/ant\nexport PATH=\ANT_HOME/bin:\$PATH" >> /etc/profile.d/ant.sh
$ souce /etc/profile.d/ant.sh
```
ㄴ 프록시 환경이라면 setproxy sudo -E 를 이용 

- 그 외 디펜던시 설치
```shell script
$ sudo yum install -y gcc gcc-c++ autoconf automake libtool pkgconfig cppunit-devel python-setuptools python-devel 
``` 
### arcus 설치 
- git checkout
```shell script
$ cd /home
$ sudo git clone https://github.com/naver/arcus.git
```
ㄴ /home 디렉토리에 arcus 를 설치하는 이유는 아커스 배포명령을 수행하면 /home/arcus 에 디폴트로 설치하기 때문  

- 빌드 
```shell script
$ sudo /home/arcus/scripts/build.sh
ARCUS BUILD PROCESS: START
--------------------------
Working directory is /home/arcus.
Detailed build log is recorded to scripts/build.log.
--------------------------
[git submodule init] .. SUCCEED
[git submodule update] .. SUCCEED
[server/config/autorun.sh] .. SUCCEED
[clients/c/config/autorun.sh] .. SUCCEED
[zookeeper/ant clean compile_jute bin-package] .. SUCCEED
[zookeeper/src/c/autoreconf -if] .. SUCCEED
[deps/libevent make clean] .. SUCCEED
[deps/libevent make] .. SUCCEED
[deps/libevent make install] .. SUCCEED
[zookeeper/src/c make clean] .. SUCCEED
[zookeeper/src/c make] .. SUCCEED
[zookeeper/src/c make install] .. SUCCEED
[server make clean] .. SUCCEED
[server make] .. SUCCEED
[server make install] .. SUCCEED
[python kazoo library install] .. SUCCEED
[python markupsafe library install] .. SUCCEED
[python jinja2 library install] .. SUCCEED
[python fabric library install] .. SUCCEED
--------------------------
ARCUS BUILD PROCESS: END
```

### 배포를 위한 ssh key 설정 
[아커스 배포 가이드](https://github.com/naver/arcus/blob/master/docs/deploying-arcus-to-multiple-servers.md) 를 보면 위에서 봤던 아커스 admin 스크립트는 ssh 를 이용해서 배포를 수행함   
ㄴ ssh 를 이용해서 여러 명령을 수행하기 위한 것
   
- root 계정의 ssh key 를 생성후 조회     
ㄴ 무조건 root 계정이어야 함   
ㄴ 배포 서버들에 /home 디렉토리에 arcus 디렉토리가 생성되어서 권한 문제가 발생 
```shell script
$ sudo ssh-keygen -t dsa -P '' -f "/root/.ssh/id_dsa"
$ sudo cat /root/.ssh/id_dsa.pub
....
```
  
- 배포할 대상서버들에 key 를 복사  
ㄴ 다른 zookeeper, memcached 서버들에 복사   
```shell script
$ sudo vi /root/.ssh/authorized_keys
# 복사한 키를 넣어줌 
```

### 설정 파일 생성
[설정 파일 생성 가이드](https://github.com/naver/arcus/blob/master/docs/arcus-cloud-configuration-file.mdhttps://github.com/naver/arcus/blob/master/docs/arcus-cloud-configuration-file.md) 에서 처럼 파일을 생성하자  
```shell script
$ ls -l /home/arcus/scripts/conf
-rw-r--r-- 1 root root  8219  5월 25 10:07 local.100node.json
-rw-r--r-- 1 root root   928  5월 25 10:07 local.10node.json
-rw-r--r-- 1 root root 12269  5월 25 10:07 local.150node.json
-rw-r--r-- 1 root root 16319  5월 25 10:07 local.200node.json
-rw-r--r-- 1 root root  4168  5월 25 10:07 local.50node.json
-rw-r--r-- 1 root root   532  5월 25 10:07 local.sample.json
-rw-r--r-- 1 root root  1037  5월 25 10:07 test.json
-rw-r--r-- 1 root root   546  5월 25 10:07 zoo.cfg
```
ㄴ 파일을 생성하고 싶지 않으면 test.json 을 수정해도 됨   
ㄴ 불필요한 local.*.json 얘네는 그냥 삭제하자     
ㄴ zoo.cfg 에는 주키퍼 관련 설정이 들어있음  
```json
{
    "serviceCode": "test_service"
  , "servers": [
        { "hostname": "cache1", "ip": ",,,", "config": { "port"   : "11211" } }
    ]
  , "config": { "threads" : "", "memlimit" : "", "connections": "" }
}
```
- 주키퍼는 service_code 별로 캐시 서버들을 관리함 
- config 는 주키퍼에 대한 설정 

### 아커스 패키지 배포
설정 파일을 기반으로 해서 아커스 패키지를 배포 
```shell script
$ cd /home/arcus/scripts
# json 은 커스텀으로 작성한 것이어도 됨 
$ sudo /bin/bash arcus.sh -z 'host1:2181, host2:2181' deploy conf/test.json
```

### 주키퍼 설치
```shell script
# 주키퍼 설정 템플릿을 각 장비에 배포 
$ sudo /bin/bash arcus.sh zookeeper init
====== Task: zk_config
...
====== Task: zk_start
...

# 주키퍼 시작 명령어 
$ sudo /bin/bash arcus.sh zookeeper start
Starting zookeeper ... STARTED

# 주키퍼 상태를 조회 
$ sudo /bin/bash arcus.sh zookeeper stat
``` 

### memcached 설치
```shell script
# 설정 파일 로드 (실제로 해당 json 배포됨)
$ sudo /bin/bash arcus.sh memcached register conf/test.json
# memcached 시작 
$ sudo /bin/bash arcus.sh memcached start test_service  

# 모든 캐시 출력
$ sudo /bin/bash arcus.sh memcached listall
# 특정 서비스 코드만 출력 
$ sudo /bin/bash arcus.sh memcached list test_service
-----------------------------------------------------------------------------------------------------
serviceCode  status  total  online  offline  created                     modified
-----------------------------------------------------------------------------------------------------
```
 

## Java client 사용
[아커스 클라우드 기본](https://github.com/naver/arcus-java-client/blob/2feec4d000/docs/01-arcus-cloud-basics.md)
아커스는 key-value 데이터 모델을 제공 (사실 memcached 기반이라서)  
하나의 key 에 multi value 를 가질 수 있는 collection 을 제공    

기본 개념 
- 서비스 코드 
- 아커스 어드민
- 캐시 key
- 캐시 item

### 서비스 코드 
아커스에서 캐시 클라우드를 구분하는 코드  
기본적으로 어플리케이션에서 사용가능한 서비스 코드는 1개 이며, 여러 서비스 코드를 사용하기 위해서는 Arcus java client 를 따로 생성하여 사용해야 함  

### 아커스 어드민 
주키퍼를 이용해서 각 서비스 코드에 해당하는 아커스 캐시 클라우드를 관리함   
특정 서비스 코드에 대한 캐시 서버 리스트를 관리  
캐시 서버 추가/삭제에 대한 캐시 서버 리스트를 최신 상태로 유지 

### 캐시 key
memcached 에 저장하는 캐시 item 을 식ㅂ려하기 위한 key
```shell script
Cache Key: [<prefix>:]<subkey>
```
- <prefix>: 캐시 key 앞에 붙는 namespace
    * prefix 단위로 cache server 에 저장된 key 들을 그룹화하여 flush 하거나 통계 정보를 볼 수 있음
    * prefix 를 생략할 수 있지만, 가급적 사용하는 것으로 ...
- delimiter: prefix 와 subkey 를 구분하는 문자로 default delimiter 는 콜론(":")
- <subkey>: 일반적으로 애플리케이션 단에서 사용할 key
