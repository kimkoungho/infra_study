# top 명령어
시스템의 상태를 간단하게 파악 가능한 명령어  
옵션 없이 입력하면 interval(3 sec) 간경으로 화면을 갱신해서 보여준다  

## 명령어 옵션
```shell script
$ man top 
``` 
* -b 옵션: 순간 정보 출력 
* -n 옵션: top 실행 주기 설정 (반복 횟수)   

## 분석 
```shell script
$ top 

top - 19:26:50 up 19 days,  7:11,  3 users,  load average: 0.39, 0.15, 0.17
Tasks: 224 total,   1 running, 223 sleeping,   0 stopped,   0 zombie
%Cpu(s):  4.5 us,  0.1 sy,  0.0 ni, 95.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16093560 total,  6793840 free,  3092128 used,  6207592 buff/cache
KiB Swap: 10485756 total, 10433788 free,    51968 used. 12604000 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 6 root      20   0       0      0      0 S   0.0  0.0   1:26.42 ksoftirqd/0
 7 root      rt   0       0      0      0 S   0.0  0.0   0:03.12 migration/0
``` 
- load average: 현재 시스템이 얼마나 일을 하는지를 나타냄    
3개의 숫자는 1분, 5분, 15분 간의 평균 실행/대기 중인 프로세스의 비율을 나타냄  
uptime 명령어로도 확인 가능  
시스템 부하를 모니터링 하는데 사용함  
숫자가 높을 수록 시스템에 부하가 있다는 것을 의미함  
load average 값은 CPU 의 코어 수를 같이 확인하며 코어 수 보다 작으면 문제가 없는 것임  

- Tasks: 프로세스의 정보를 출력하는 곳  

- KiB Mem : 메모리 정보를 출력
- KiB Swap : 페이지 swap 정보 출력 

- PID: 프로세스 ID
- USER: 명령어를 실행한 유저
- PR: 프로세스의 우선순위
- VIRT: 가상 메모리 사용량 (SWAP + RES)
- RES: 현재 페이지가 상주하고 있는 크기 (Resident Size)
- SHR: 분할된 페이지, 프로스세에 의해 사용된 메모리를 나눈 메모리의 총합
- S: 프로세트 상태 (작업중, I/O 대기, 유휴상태 등)
- CPU: 프로세스가 사용하는 CPU 사용률
- MEM: 프로스세스가 사용하는 메모리 사용률
- TIME: 프로세스가 시작된 이후 경과된 총 시간 
- COMMAND: 실행된 명령어 

## top 명령어 실행후 명령어
- shift + t : 실행된 시간이 큰 순서로 정렬 
- shift + m : 메모리 사용량이 큰 순서로 정렬
- shift + p : cpu 사용량이 큰 순서로 정렬 
- k : 프로세스 종료
    * k 입력후 종료할 PID 를 입력 
    * signal 을 입력하라고 표시하면 9 를 넣어줌 (kill -9) 
- space : refresh
- u : 입력한 유저 소유의 프로세스만 표시 

## ps vs top
ps 는 명령어를 수행한 시점에 프로세스의 정보를 출력   
top 은 프로세스들의 정보 + cpu + 메모리 등을 일정 주기로 합산하여 출력  
ㄴ 모니터링 할 때 좋음 


## 래퍼런스 
- [CUBRID 게시글](https://www.cubrid.com/tutorial/3794195)
- [zzsza 블로그](https://zzsza.github.io/development/2018/07/18/linux-top/)