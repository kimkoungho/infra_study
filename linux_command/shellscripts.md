# 조건문

## if 
if 문 옵션
-b 파일 : 파일이 블럭 장치 파일이면 true
-c 파일 : 파일이 문자 장치 파일이면 true
-d 파일 : 파일이 디렉토리이면 true
-e 파일 : 파일이 존재하면 true
-f 파일 : 파일이 정규파일이면 true
-L 파일 : 파일이 심볼릭 링크이면 true
-p 파일 : 파일이 named 파일이면 true
-S 파일 : 파일이 소켓이면 true
-r 파일 : 파일이 읽기가능하면 true
-s 파일 : 파일의 크기가 0보다 크면 true
-w 파일 : 파일이 쓰기 가능하면 true
-x 파일 : 파일이 실행 가능하면 true
-z 문자열 : 문자열의 길이가 0이면 true
-n 문자열 : 문자열의 길이가 0이 아니면 true
파일1 -nt 파일2 : 파일1 이 파일2 보다 새로운 파일이면 true
파일1 -ot 파일2 : 파일1 이 파일2 보다 오래된 파일이면 true
파일1 -ef 파일2 : 파일1 과 파일2 가 같은 파일이면 true
 

## case
- test.sh
```shell script
val=$1
case $val in
   "start")
      echo "시작"
      ;;
    "stop")
      echo "종료"
      ;; 
```
- ) 는 조건을 명시
- ;; 는 break 문과 같음 


## shift
bash 인수목록에서 시작 부분에서 인수를 제거하는 내장 기능  
예를 들어 $1, $2, $3 를 사용할 수 있는 경우 shift 사용시 $1 은 $2 가 될 것이다 
 
## 인수 
- $0
https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_$0
명령 라인에서 실행 시 쉘명
스크립트에서 실행 시 실행된 쉘 스크립트 경로를 포함한 파일명

- $1 : 처음 파라미터 
- $2 : 두번째 파라미터
- $# : 파라미터 개수
- $@ : 파라미터 전체
- $$ : 현재 쉘 프로세스 ID