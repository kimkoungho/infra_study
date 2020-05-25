# 아커스 설치 및 사용법 

## 아커스 설치

### 아커스 어드민 스크립트
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
- 현재 zookeeper 에 등록된 모든 캐시들에 memcached 를 설치 

```shell script
./arcus.sh [-z <zklist>] quicksetup <conf_file>
```
- deploy, zookeeper init & start, memcached register & start 작업을 한번에 수행 

### 실제로 설치 해보기 


