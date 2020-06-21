# rsync
rsync 는 Remote Sync 의 약자로 두 호스트 사이에서 file & directory 동기화를 위한 unix 기반 유틸리티  
전송시 네트워크 대역폭을 최소화하는 delta encoding 알고리즘이 구현되어 있어서 rcp 나 scp 보다 훨씬 빠르고 효율적으로 동기화가 가능     
  
rsync 는 클라이언트와 서버 프로그램이 모두 포함되어 있으며 서버로 구동될 경우 TCP 의 873 포트를 사용  
서버로 구동시 ssh, rsh 기반에서 동작 가능하기 때문에 rsync 를 사용하면 추가적인 방화벽 오픈 없이 사용가능  
ㄴ (ssh 추천)  

## format
```shell script
$ rsync options source destination
```
주요 옵션   
-v : verbose 자세한 정보 출력  
-r : 재귀적으로 하위 디렉토리 까지 복사  
-t : 타입스탬프 보존 (복사할 파일의 시간을 복사)
-l : 심볼릭 링크 보존    
-p : permission 보존   
-g : 그룹 보존  
-o : 소유자 보존(root 만 가능)  
-D : device special 파일 보존  
-a : active mode 로 위에 있는 -r, -t, -l, -p, -g, -o, -D 옵션을 포함한다  
-z : 데이터 압축  
-h : human-readable  
-u : 복사할 파일 중 같은 이름이 존재할 경우 덮어쓰기를 수행하지 않음
-d : 디렉토리 구조만 복사하는 옵션
-e : ssh 와 연결할 포트를 지정할 수 있음      
--delete: 원본 서버에 없는 파일 대상 서버에서 삭제 
--delete-excluded: 원본 서버에서 삭제하지 않을 파일을 지정 
--exclude: 대상 서버로 복사하지 않을 파일을 지정
--include: 대상 서버로 복사할 파일을 지정    
--rsh: 원격 shell 사용 
--progress: 전송시 진행내역 출력  
  
기본적으로 특정 디렉토리의 모든 파일을 복사한다면 -az 옵션만으로도 충분함  
ㄴ 한가지 주의할 점은 rsync 옵션들은 os 에 설치된 rsync 버전별로 차이가 있음    
ㄴ 만약 원하는 옵션이 없다면 rsync 버전 업데이트를 해보길 추천    

## 사용법
- 로컬 파일을 로컬로 복사   
```shell script
$ tree test
test
├── dir1
└── dir2

# 복사 
$ rsync -a test test_dest

# 확인 
$ tree test_dest/
test_dest/
└── test
 ├── dir1
 └── dir2
```
ㄴ 주의할 점은 디렉토리 자체를 복사한다는 것  

- 특정 디렉토리의 내부만 복사하기 
```shell script
$ rsync -a test/ test_dest

# 확인 
$ tree test_dest/
test_dest/
├── dir1
└── dir2
```
  
이 예제에서는 대상 호스트를 지정하지 않았는데, rsync 는 단일 호스트에서 파일을 복사할 때에도 사용할 수 있음  
  
  
- 로컬 파일을 원격 서버로 복사   
```shell script
$ rsync -avz /home/local/ [유저]@[호스트]:/home/remote/
```
ㄴ 사용법은 scp 와 거의 유사함  

- 원격 파일을 로컬로 복사 
```shell script
$ rsync -avz [유저]@[호스트]:/home/remote/ /home/local/ 
``` 

- 원하는 확장자 파일만 복사  
```shell script
$ rsync -zarv --include="*/" --include="*.sh" --exclude="*" /home/local [유저]@[호스트]:/home/remote
```
include 에 모든파일(*) 을 지정해서 모든파일을 포함시키고 그곳에서 shell(.sh) 을 포함 시킨뒤 모든 파일(*)을 exclude 하면 됨  
  
- symbolic link  
    * link 처리 
    ```shell script
    $ rsync -avz --link /home/local [유저]@[호스트]:/home/remote
    ```
    * 참조하는 원본 복사 (link 대상파일)
    ```shell script
    $ rsync -avz -L /home/local [유저]@[호스트]:/home/remote
    ```
    * link 파일 제외 
    ```shell script
    $ rsync -avz --no-links /home/local [유저]@[호스트]:/home/remote
    ```

## copy vs rsync
위에 예제에서 rsync 로 로컬 파일을 로컬로 복사하는 예제가 있었다  
그럼 일반적으로 사용하는 cp(copy) 명령어와 차이점이 뭘까 ?  
  
[copy_vs_rsync_stackoverflow](https://stackoverflow.com/questions/6339287/copy-or-rsync-command) 글에 따르면, rsync 가 업데이트 된 부분 만 복사해서 copy 보다 성능이 좋다고 한다     
예를들어 test.txt 파일이 자주 수정되어 복사하는 경우에는 rsync 가 변경된 부분만을 감지해서 복사하기 때문에 성능이 더 좋다   
하지만, 특정 디렉토리 하위에서 파일이 추가는 일어나지만 파일 내용 업데이트 되지 않는 경우에는 cp -u 가 더 효율적이라고 한다      


## Reference
- [system-admin rsync 사용법](https://www.lesstif.com/pages/viewpage.action?pageId=12943658)
- [copy_vs_rsync_stackoverflow](https://stackoverflow.com/questions/6339287/copy-or-rsync-command)
- [rsync_include_extension_stackoverflow](https://stackoverflow.com/questions/11111562/rsync-copy-over-only-certain-types-of-files-using-include-option)
