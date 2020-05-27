# docker 관련 이슈 내용 정리하자 ! 

- 도커 네트워크 대역과 실제 서버 네트워크 대역이 같으면 충돌이 발생
ㄴ 도커 네트워크 대역을 수정해야 함 
 
```shell script
# 라우팅 룰 확인 
$ sudo route -n | grep 172

# 도커 정지 
$ sudo service docker stop
# 도커 네트워크 인터페이스 제거 
$ sudo ip link del docker0

# 충돌이 나지 않는 대역 추가 
$ sudo vi /etc/docker/damon.json
{"bip": "10.~"}
# 재시작
$ sudo docker service restart
# 확인 
$ sudo route -n | grep 172  
``` 

 