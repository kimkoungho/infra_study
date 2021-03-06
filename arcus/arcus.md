# arcus
네이버가 만든 메모리 캐시 클라우드   
DB 앞단에 hot-spot 성격의 데이터를 캐싱   
  
멤캐시를 확장한 솔루션  
기본적인 데이터 타입에 대한 캐싱 기능에 부가하여 List, Set, B+ Tree 같은 데이터 콜랙션 처리가 가능  

아커스 클러스터 정보  
- cluster name
- cache node list 
  
cache node list 변경을 아커스 클라이언트에게 실시간으로 알리기 위해서 주키퍼를 사용  

arcus memcached

실제 데이터가 저장되는 캐시 역할 
아커스 memcached 서버가 start/stop 될 때, zookeeper 에 notify
문제가 발생한 아커스 memcached 는 스스로 중지
arcus zookeeper

zookeeper 는 service_code 를 이용하여 memcached 그룹을 관리함
service_code 로 묶인 memcached 그룹의 변화가 생기면 아커스 클라이언트에 notify
ex) memcached 서버 중지 
아커스 memcached 서버 주기적으로 확인
arcus client

arcus memcached

실제 데이터가 저장되는 캐시 역할 
아커스 memcached 서버가 start/stop 될 때, zookeeper 에 notify
문제가 발생한 아커스 memcached 는 스스로 중지
arcus zookeeper

zookeeper 는 service_code 를 이용하여 memcached 그룹을 관리함
service_code 로 묶인 memcached 그룹의 변화가 생기면 아커스 클라이언트에 notify
ex) memcached 서버 중지 
아커스 memcached 서버 주기적으로 확인
arcus client



## 구성도
아커스 서버 (memcached)
- 캐시 데이터 저장소
- 아커스 서버 프로세스가 start/stop 되면 주키퍼에 통보
- 문제가 발생한 아커스는 스스로 중지

아커스 주키퍼 (admin)
- 서비스 코드 별로 아커스 서버 목록을 관리하고 변경이 있을때 클라이언트에 통지 (서버 노드 추가/제거)
- 아커스 서버와의 세션이 만료되면 아커스 서버 목록에서 해당 서버을 제거
- 서비스 코드를 이용해서 memcached (아커스 서버) 서버를 관리 
- 주키퍼는 아커스 서버들이 살아 있는지 주기적으로 확인하여 아커스 클라이언트에 알림 

아커스 클라이언트
- 주키퍼로부터 서비스 코드에 할당된 사용가능한 아커스 서버 목록을 받아옴
ㄴ 주키퍼 노드를 watch 하고 있다가 추가/제거 여부를 물어봄
- 아커스 서버 목록의 변경을 지속적으로 감시 (주키퍼 watcher)
- 아커스 서버 목록 변경시, 최신 서버 목록 기반으로 요청을 리해싱
- Java, c 아커스 클라이언트를 제공 
ㄴ [Java client](https://github.com/naver/arcus-java-client/tree/2feec4d000ab13f956ea19d9b9c2cc67b12713b3)
ㄴ [c client](https://github.com/naver/arcus-c-client/tree/561cc85eada06b0ca895fbd2a39db737d40e38ff)

[!No Image](images/arcus-architecture.png)

## 동작방식
### 아커스 서버가 서비스 클러스터 노드에 참여 
1. 아커스 서버의 IP:PORT 를 사용하여 자신이 서비스 되어야 하는 코드를 조회 (IP 인덱싱)
2. 서비스 코드 노드 아래 자신의 IP:PORT 를 ephemeral node 로 생성
ㄴ ephemeral node: 주키퍼와 세션이 유효한 동안에만 존재하는 노드
ㄴ 클라우드에 참여하는 서버 정보 관리를 위해 주키퍼의 znode 구조를 이용함 

### 아커스 서버가 서비스 클러스터 노드에 연결
1. 서비스 코드에 등록된 아커스 서버 목록을 조회
2. 아커스 서버로 접속을 시도
3. 자신의 서비스 코드의 변경 사항 감시 
ㄴ 자식 노드에 변경이 생기면 해당 내용을 통지받아 서버 목록을 갠신
ㄴ 서비스 코드명을 가진 노드를 주기적으로 watch

### 아커스 서버가 서비스 노드에서 탈퇴 
1. 주키퍼 서버에 ephemeral znode 를 생성하여 connection timeout, disconnect, 세션 만료가 되면 자동으로 znode 삭제
2. 네트워크/서버/OS 등은 잘 동작하고 있지만 아커스가 응답하지 못하는 상황을 감지하기 위해 주키퍼 hearbeat 를 할 때 자기자신에게 dummy 요청을 하여 응답이 오는지 확인
3. 클라이언트는 znode 의 변경사항을 감지하여 변경이 있으면 변경된 내용으로 서버 목록을 갱신

 

## 래퍼런스
[네이버 arcus git](https://github.com/naver/arcus/blob/master/docs/deploying-arcus-to-multiple-servers.md)
[JaM2in pdf](http://www.jam2in.com/resources/ARCUS_brochure.pdf)
[티스토리](https://12bme.tistory.com/549)








